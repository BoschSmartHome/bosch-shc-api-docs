# Zigbee Routing Information

Community-documented notes on an undocumented local API endpoint that exposes the Zigbee mesh
route between a battery-powered end device and the Smart Home Controller. Discovered by
decompiling the official Bosch Android app (version 10.33.4335) and cross-verified against real
Bosch Smart Home Controller hardware (firmware current as of 2026-07). This endpoint is **not**
part of the official OpenAPI spec published in this repository.

## Purpose

This is the data source behind the app's device-details "Routing" display and the
communication-test wizard's route view. It lets you inspect the actual Zigbee mesh path (and the
link quality of each hop) that a given device currently uses to reach the controller — useful for
diagnosing weak-signal or intermittent-connectivity issues without relying on the app.

## Request

```
GET /smarthome/zigbee/routinginfo/{deviceId}
```

- Port `8444`, same mTLS client-certificate authentication as all other `/smarthome` endpoints.
- `{deviceId}` must be URL-encoded (for example `hdm%3AZigBee%3A0123456789abcdef`), the same as
  the encoding rule already documented for child device IDs containing `#` (see `CHANGELOG.md`).
- Only valid for devices whose id has the `hdm:ZigBee:` prefix. Calling it for a non-Zigbee device
  (e.g. an `hdm:HomeMaticIP:...` or virtual device id) is not supported.

Example (host/device id anonymized):

```
GET https://{shcIp}:8444/smarthome/zigbee/routinginfo/hdm%3AZigBee%3A0123456789abcdef
api-version: {{api_version}}
```

## Response schema

The response mirrors the APK class `com.bosch.sh.common.model.zigbee.ZigbeeRoutingInformationData`:

| Field | Type | Description |
|---|---|---|
| `device` | string | The queried device's id (`hdm:ZigBee:...`), echoed back. |
| `aggregatedQuality` | enum | Overall route quality — the worst-case (or otherwise aggregated) quality across all hops in `route`. See enum values below. |
| `route` | array | Ordered list of hops from the device toward the controller. The first element is always the device itself; the last element is effectively the controller side of the final hop. Empty if `aggregatedQuality` is `NO_CONNECTION`. |
| `route[].deviceId` | string | The Zigbee device id at this hop. |
| `route[].quality` | enum | Link quality for this specific hop. Same enum as `aggregatedQuality`. |

### Quality enum (complete, from APK)

- `GOOD`
- `MEDIUM`
- `BAD`
- `NO_CONNECTION`
- `DEVICE_NOT_INITIALIZED`
- `NOT_SUPPORTED`
- `UNKNOWN`

## Example exchanges

All device ids below are anonymized placeholders.

### Healthy direct connection

Device talks straight to the controller, no intermediate router hop:

```json
{
  "device": "hdm:ZigBee:0123456789abcdef",
  "aggregatedQuality": "GOOD",
  "route": [
    {"deviceId": "hdm:ZigBee:0123456789abcdef", "quality": "GOOD"}
  ]
}
```

### Multi-hop through a mains-powered router

A battery-powered device (e.g. a door/window contact) routes through a mains-powered device
(e.g. a Smart Plug Compact) that acts as a Zigbee router:

```json
{
  "device": "hdm:ZigBee:0123456789abcdef",
  "aggregatedQuality": "GOOD",
  "route": [
    {"deviceId": "hdm:ZigBee:0123456789abcdef", "quality": "GOOD"},
    {"deviceId": "hdm:ZigBee:fedcba9876543210", "quality": "GOOD"}
  ]
}
```

This was verified with two real cases: a door/window contact routing through a Smart Plug
Compact, and a radiator thermostat routing through a different Smart Plug Compact.

### No connection

```json
{
  "device": "hdm:ZigBee:0123456789abcdef",
  "aggregatedQuality": "NO_CONNECTION",
  "route": []
}
```

## Relationship to the `Routing` device service

Do not confuse this endpoint with the `Routing` **service** (`RoutingState`, values `ENABLED` /
`DISABLED`) exposed under `GET /smarthome/devices/{id}/services/Routing/state`. That service only
exists on the classic Smart Plug (`PSM`) and lets you explicitly toggle whether that specific
device participates in Zigbee mesh routing.

The Smart Plug Compact and Micromodules have no such service at all — their router role is
always-on and cannot be toggled. For these devices, `GET /smarthome/zigbee/routinginfo/{deviceId}`
is the only way to observe whether (and through which router) a battery device's traffic is
actually flowing, since there is no configuration to inspect, only the resulting route.

## Verification

Confirmed working end-to-end against a real Bosch Smart Home Controller (mTLS client
authentication, port 8444) on 2026-07-10, cross-checked against the response shape decompiled from
the official Bosch Android app (version 10.33.4335).
