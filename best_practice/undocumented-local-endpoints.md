# Undocumented Local API Endpoints

This document is a narrative companion to a set of new OpenAPI files added to
this repository. It describes a handful of local REST API endpoints of the
Bosch Smart Home Controller that exist and work today, but were not covered
by the official OpenAPI generator output.

These endpoints were traced primarily by decompiling the official Bosch
Android app; most were then confirmed end-to-end against a real Bosch Smart
Home Controller. Each section below states its confidence level explicitly:

- **Live-confirmed** - exercised against a real Bosch Smart Home Controller
  and observed to behave as documented.
- **Spec-only** - the request/response shape is transcribed from decompiled
  app code (Kotlin data classes or equivalent), but no capable hardware was
  available to exercise it. Treat these as a best-effort starting point, not
  a guaranteed contract, and please report back if you find a discrepancy on
  real hardware.

As with the rest of this repository, by using any of the endpoints below you
accept and agree to be bound by our [Terms and
Conditions](https://github.com/BoschSmartHome/bosch-shc-api-docs#terms-and-conditions),
and this documentation is subject to the [Creative Commons
Attribution-NonCommercial-NoDerivatives 4.0 International Public
License](https://creativecommons.org/licenses/by-nc-nd/4.0/legalcode).

## 1. Firmware management

**Confidence: live-confirmed.**

Two device-agnostic endpoints and one controller-level endpoint let a client
observe and drive firmware updates without polling every Device's individual
service data:

- `GET devicemanagement/firmware/{deviceId}` returns a bare JSON string
  describing where a single Device is in its firmware lifecycle (e.g.
  `"UpToDate"`, `"AwaitingActivation"`, `"UpdatePending"`). Devices without a
  firmware capability - virtual devices, or the Controller's own root Device
  - respond `404`; this is expected and simply means "no firmware state to
  report", not an error.
- `PUT devicemanagement/firmware/{deviceId}/activate` (no request body)
  triggers installation of a pending update for one Device. This was
  confirmed end-to-end on a real TRV_GEN2 (Thermostat II): the firmware state
  moved through `AwaitingActivation` -> `UpdatePending` ->
  `UpToDateAwaitingUserInteraction` over roughly 90 seconds after the call.
- `POST rootdevices/startSoftwareUpdate` (no request body) triggers the
  Controller's own firmware update, the local-API equivalent of the "Update
  now" button for the Controller itself in the Bosch app.

See `openapi/DeviceManagement-local-openapi-v3.yml`.

## 2. Automation rule enable/disable and delete

**Confidence: live-confirmed.**

The official spec already documents listing and reading automation rules
(`GET /automation/rules`, `GET /automation/rules/{automationId}`) and
manually firing one (`PUT /automation/rules/{automationId}/trigger`) in
`openapi/MainResources-local-openapi-v3.yml`. Two further operations on the
very same `/automation/rules/{automationId}` path exist:

- `PUT /automation/rules/{automationId}` - a full-body PUT that updates the
  rule, most commonly used to flip its `enabled` field on or off. The
  Controller expects the complete `Automation` body back (read the current
  rule, mutate the field you want to change, PUT the whole thing), the same
  read-modify-write pattern used elsewhere in this API.
- `DELETE /automation/rules/{automationId}` - permanently deletes the rule.
  No request or response body.

Both were added as additional operations under the existing path entry in
`openapi/MainResources-local-openapi-v3.yml`, rather than a new file, since
the resource itself is already documented there.

## 3. Intrusion-system read-only discovery

**Confidence: live-confirmed** (all six responses were observed against a
real Controller); **the exact field shape beyond `id` is not confirmed** for
several of them, see below.

The Intrusion Detection Alarm System exposes several read-only discovery
endpoints beyond the already-documented `/intrusion/states/system` and
`/intrusion/actions/*`:

- `GET /intrusion/profiles` - lists the system's configuration profiles
  (e.g. "Full Protection", "Partial Protection", "Individual" - the same
  profiles referenced by the existing `ArmRequest` enum on the `arm` action).
- `GET /intrusion/profiles/{profileId}` - detail for a single profile.
- `GET /intrusion/states/profiles` - the current arming state of every
  profile.
- `GET /intrusion/endpoints/alarm/actuators` - alarm-actuator endpoints
  (sirens etc.) known to the system.
- `GET /intrusion/endpoints/alarm/triggers` - alarm-trigger endpoints (door
  contacts, motion detectors, etc.).
- `GET /intrusion/endpoints/reminder/actuators` - reminder-actuator
  endpoints (entrance-delay chimes etc.).

The response envelope (array vs. single object) is confirmed live for all
six. The exact field set of each element beyond `id` was not independently
confirmed, so the corresponding OpenAPI schemas are intentionally kept
generic (`type: object` with no fixed property list, or a minimal
`IntrusionProfile` schema with just `id`/`name`) rather than guessing at
undocumented fields. See `openapi/IntrusionDetectionSystem-local-openapi-v3.yml`.

## 4. Thermostat regulation algorithm

**Confidence: live-confirmed** (the 404 path was exercised against every
real thermostat/valve available for testing; none of them support this
capability, so the 200 response body itself was not observed on real
hardware, only transcribed from the decompiled app).

- `GET thermostat/regulation/{deviceId}/config` returns
  `{"id": "...", "algorithmUsed": "INTERNAL"}` (or `"CUSTOM"`) for
  thermostat-capable Devices that support this feature, `404` for those that
  don't.
- `PUT thermostat/regulation/{deviceId}/config` writes the same shape to
  change which algorithm the Device's local firmware uses to regulate
  temperature.

See `openapi/ThermostatRegulation-local-openapi-v3.yml`.

## 5. Temperature-drop service

**Confidence: live-confirmed across 12 real Rooms.**

Each Room can have an anti-frost/window-open temperature-drop service that
temporarily lowers its heating setpoint by a configured amount:

- `GET climate/temperaturedropservice` lists every Room with this
  capability.
- `GET climate/temperaturedropservice/{roomId}` returns one Room's
  configuration, a `{"id", "configuration": {"enabled", "dropTemperature"}}`
  shape (plus additional server-derived `supports*`/`requires*` capability
  flags observed live but not meant to be written).
- `PUT climate/temperaturedropservice/{roomId}` writes the same shape back
  (read-modify-write - only the nested `configuration` object is meant to be
  edited).

See `openapi/TemperatureDropService-local-openapi-v3.yml`.

## 6. Hydraulic balancing

**Confidence: spec-only, not independently verified against real hardware.**

`GET climate/hydraulicbalancing`, `GET
climate/hydraulicbalancing/{configId}`, and `PUT
climate/hydraulicbalancing/{configId}` expose per-heating-circuit hydraulic
balancing configurations. No hydraulic-balancing-capable Device was
available for live testing - the test installation used to trace this API
responds `503` for all three endpoints, confirming the *paths* exist but not
the response shape. The schema in
`openapi/HydraulicBalancing-local-openapi-v3.yml` was transcribed from the
decompiled Bosch Android app's Kotlin data classes only and is kept
intentionally minimal (just `id`) beyond what could be confirmed. Please
report back if you have capable hardware and can confirm or correct the
shape.

## 7. Comfort-zone templates

**Confidence: spec-only, not independently verified against real hardware.**

`GET airquality/comfortzone/templates/{sensorId}` and `PUT
airquality/comfortzone/templates/{sensorId}/custom` let a client read and
save comfort-zone templates for an air-quality sensor such as Twinguard. No
Twinguard (or other air-quality) Device was available for live testing, so
- like hydraulic balancing above - the schema in
`openapi/ComfortZoneTemplates-local-openapi-v3.yml` is a best-effort
transcription of the decompiled app rather than a live-confirmed contract.
Please report back if you can confirm or correct the shape on real hardware.

## 8. Missing fields on the open doors/windows response

**Confidence: live-confirmed.**

`GET /doors-windows/openwindows` is already documented in
`openapi/MainResources-local-openapi-v3.yml`, returning a `Windows` object
with `allDoors`/`openDoors`/`unknownDoors`/`allWindows`/`openWindows`/
`unknownWindows`/`allOthers`/`openOthers`/`unknownOthers`. A live call
against a real Controller also returned three further arrays not previously
in the schema: `bypassedDoors`, `bypassedWindows`, and `bypassedOthers` -
entries currently bypassed, most commonly via the Intrusion Detection Alarm
System. These three fields were added to the existing `Windows` schema in
`openapi/MainResources-local-openapi-v3.yml`; no new file was needed since
the schema was otherwise already correct.
