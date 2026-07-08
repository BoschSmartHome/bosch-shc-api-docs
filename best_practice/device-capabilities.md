# Device Capabilities and Ports

Community-maintained notes on which services a given device type actually exposes on the local
API, and which port to use for which operation. Compiled from direct queries against real Bosch
Smart Home Controller hardware (Gen1 and Gen2, various device types) by maintainers of the
[boschshcpy](https://github.com/tschamm/boschshcpy) Python library and the
[boschshc-hass](https://github.com/tschamm/boschshc-hass) Home Assistant integration, which build
on top of this API.

## Ports

The local API is split across three distinct ports. Mixing them up is one of the most common
sources of confusion in this repository's issues:

| Port | Purpose | Auth |
|------|---------|------|
| `8443` | Client **registration/pairing** — `POST /smarthome/clients` | Systempassword header, no client cert yet |
| `8444` | Main local REST API and Long Polling — `https://{shcIp}:8444/smarthome/...` | mTLS (client cert + key issued during registration) |
| `8446` | Public, unauthenticated **info-only** endpoint — `https://{shcIp}:8446/smarthome/public/information` | none |

Once a client is registered via `8443`, all normal device/service calls (including Long Polling)
go through `8444` with the client certificate. `8446` is read-only and only exposes basic
controller information (model, firmware, etc.) — it cannot be used to read or set device state.

## ChildProtection / child lock availability by device type

The Bosch Smart Home app shows a "child lock" toggle for many device types, but at the local API
level this is implemented two different ways — and for one device family it isn't implemented at
all:

| Device model | Local API mechanism |
|---|---|
| `BSM_LIGHT_SWITCH` | `ChildProtection` service |
| `MICROMODULE_RELAY` | `ChildProtection` service |
| `MICROMODULE_SHUTTER` | `ChildProtection` service |
| `MICROMODULE_BLIND` | `ChildProtection` service |
| `MICROMODULE_DIMMER` | `ChildProtection` service |
| `TRV`, `TRV_GEN2` (thermostats) | `Thermostat` service's `childLock` field (`ON`/`OFF`) — **not** a separate `ChildProtection` service |
| `THB` (room thermostat) | Same `Thermostat`-service mechanism as above |
| `PLUG_COMPACT`, `PLUG_COMPACT_DUAL` | **No child-lock mechanism at all.** `GET /smarthome/devices/{id}/services` returns only `PowerMeter`, `PowerSwitch`, `PowerSwitchConfiguration`, `PowerSwitchProgram`, `CommunicationQuality` — confirmed by querying all 5 PLUG_COMPACT devices on a real controller. The app's child-lock toggle for this device type is not backed by a local-API service; see [#126](https://github.com/BoschSmartHome/bosch-shc-api-docs/issues/126). |

If you're implementing a third-party integration: check the device's `deviceModel` first, then
branch on whether to look for a `ChildProtection` service or a `Thermostat.childLock` field —
assuming one universal mechanism across all device types will silently miss cases.
