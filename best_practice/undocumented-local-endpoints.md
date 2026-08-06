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

## 9. RoomClimateControl write ordering: `summerMode` must be cleared before `operationMode`

**Confidence: live-confirmed.**

`RoomClimateControlService` (see `openapi/ClimateControl-local-openapi-v3.yml`)
exposes `summerMode`, `coolingMode`, and `operationMode` as independently
writable fields, each set via its own `PUT` call - there is no combined,
atomic write for "change HVAC mode". This is already documented as three
separate fields, but the Controller enforces an undocumented **write-order
dependency** between them that isn't visible from the schema alone:

- Writing `PUT operationMode` while `summerMode` is still `true` is
  **rejected outright** with HTTP 400:
  ```json
  {"@type":"JsonRestExceptionResponseEntity","errorCode":"WRONG_THERMOSTAT_GROUP_MODE","statusCode":400}
  ```
- `summerMode` must be cleared (`PUT summerMode: false`) **first**, and
  only then does a subsequent `PUT operationMode` succeed.

Practical consequence for any client trying to switch a Room from "off"
(`summerMode=true`) directly to a specific heating mode (`operationMode`):
the two writes cannot be reordered or combined to avoid a momentary
intermediate state where `summerMode=false` but `operationMode` still holds
its old value - the Controller requires exactly that sequence, and a client
that tries to write `operationMode` first (e.g. to avoid a UI flicker) will
have the write rejected rather than merely see the fields settle in a
different order. Confirmed by a real Home Assistant integration's own
regression: reordering the writes to avoid a cosmetic activity-log
side-effect broke real HVAC-mode changes in production against a live
Controller.

`coolingMode` was not separately confirmed to share this exact dependency,
but the safe, tested-working order for any full mode transition is:
`summerMode=false` first, then `coolingMode`/`operationMode` as needed.

## 10. Creating Automations and UserDefinedStates

**Confidence: live-confirmed.**

The official spec documents listing/reading Automations and UserDefinedStates
and updating/deleting Automations (see section 2 above), but not *creating*
either resource. Both support a genuine `POST`:

- `POST /automation/rules` - creates a new Automation. Body is a full
  `Automation` object with `id` left blank (empty string); the Controller
  assigns the id server-side and echoes the created object, including the
  new id, in the response.
- `POST /userdefinedstates` - creates a new UserDefinedState (a virtual
  on/off flag other Automations can read or set). Same pattern: blank `id`
  in the request, server-assigned id in the response.

Traced from the official Bosch Android app
(`RestRequests.getAutomationCreateRequest` /
`RestRequests.createUserDefinedState`, both building on the same
`buildPostRequest`/`ConnectorLibraryRequest` machinery as every other local
write in this API - not a separate cloud-only code path) and confirmed with
a live create-then-delete round trip against a real Controller for both
resources. Added as `post:` operations under the existing path entries in
`openapi/MainResources-local-openapi-v3.yml`.

### Automation trigger/condition/action type registry (partial, live-observed)

The `Automation` schema's `automationTriggers`/`automationConditions`/
`automationActions` fields are documented only as raw, untyped `object`
arrays - there is no published schema for what goes inside one. Each entry
is actually a `{"type": "<TypeName>", "configuration": "<JSON-string>", ...}`
envelope, where `configuration` is itself a **JSON-encoded string** (not a
nested object) holding a type-specific payload. `automationActions` entries
additionally carry a `delayInSeconds` integer (0 for "immediately"; used
e.g. to auto-reset a UserDefinedState back to `INACTIVE` some seconds after
setting it `ACTIVE`).

The type registry is large (dozens of trigger/condition/action types) and
not documented anywhere; below are only the entries actually observed live
(via a real Controller's existing Automations, created through the Bosch
app) or decompiled with a concrete field-level example. Treat this as a
seed, not a complete list.

**Triggers:**

- `KeypadButtonPressTrigger` - fires on a physical button press (a
  Universal Switch, Light/Shutter Control II detached button, etc.):
  ```json
  {"type": "KeypadButtonPressTrigger",
   "configuration": "{\"deviceId\":\"<device id>\",\"keyName\":\"UPPER_BUTTON\",\"keyCode\":1,\"buttonEvent\":\"PRESS_SHORT\"}"}
  ```
  `buttonEvent` observed as `PRESS_SHORT` and `PRESS_LONG`; `keyName` values
  observed include `UPPER_BUTTON`/`LOWER_BUTTON` and `BUTTON_1`/`BUTTON_2`
  (device-dependent labeling), paired with `keyCode` `1`/`2` respectively.
- `UserDefinedStateTrigger` - fires when a UserDefinedState changes:
  ```json
  {"type": "UserDefinedStateTrigger",
   "configuration": "{\"stateChange\":\"TO_ACTIVE\",\"stateId\":\"<userDefinedState id>\"}"}
  ```
  `stateChange` observed as `TO_ACTIVE` and `TO_INACTIVE`.
- `ShutterContactTrigger` / `ShutterContactReminderTrigger` - fire on a
  door/window contact's open/close state (the latter after a
  `remindAfterSeconds` delay in the still-open/closed state):
  ```json
  {"type": "ShutterContactTrigger",
   "configuration": "{\"shutterContactId\":\"<device id>\",\"triggerState\":\"ON_OPEN\"}"}
  {"type": "ShutterContactReminderTrigger",
   "configuration": "{\"shutterContactId\":\"<device id>\",\"triggerState\":\"CLOSED\",\"remindAfterSeconds\":900}"}
  ```
- `ShutterContactButtonPressTrigger` - fires on the physical pushbutton of a
  Door/Window Contact II (SWD2/SWD2_PLUS/SWD2_DUAL), which has no Keypad
  service and is otherwise invisible to the local API:
  ```json
  {"type": "ShutterContactButtonPressTrigger",
   "configuration": "{\"shutterContactId\":\"<device id>\",\"buttonPressState\":\"ON_SHORT_PRESS\"}"}
  ```
  `buttonPressState` observed as `ON_SHORT_PRESS` and `ON_LONG_PRESS`. Live-
  confirmed 2026-08-06: created via the official Bosch app (mapping a real
  SWD2_DUAL button to a Scenario), read back via `GET /automation/rules`,
  then deleted again via `DELETE /automation/rules/{id}` (see §10 above).
  Resolves the blocker tracked in hass#245/#342/#376 — the `GET/PUT
  /keypads/{deviceId}/configuration` endpoint speculated about in an earlier
  investigation does NOT carry this (all 3 of Thomas's SWD2 devices return
  `{"assignments":[],...}` there regardless); the real mechanism is this
  ordinary Automation-rule trigger, same shape as `KeypadButtonPressTrigger`.
- `DailyTimerTrigger` - fires at a fixed time of day:
  `{"type": "DailyTimerTrigger", "configuration": "{\"time\":\"23:00\"}"}`.
- `AstroTimeEventTrigger` - fires relative to sunrise/sunset:
  `{"type": "AstroTimeEventTrigger", "configuration": "{\"eventType\":\"SUNSET\",\"offsetInMinutes\":60}"}`.

**Conditions:** `IntrusionDetectionSystemArmedDisarmedCondition`
(`conditionState`: `SYSTEM_DISARMED`), `AstroTimeCondition`
(`astroTimeConditionType`: `NIGHTTIME`/`DAYTIME`, `startOffsetInMinutes`),
`SmartPlugOnOffCondition` (`smartPlugId`, `conditionState`: `ON`/`OFF`),
`ShutterContactCondition` (`shutterContactId`, `conditionState`:
`OPEN`/`CLOSED`).

**Actions:**

- `UserDefinedStateAction` - sets a UserDefinedState to an absolute value
  (no relative "toggle" - compute the inverse client-side if needed):
  ```json
  {"type": "UserDefinedStateAction", "delayInSeconds": 0,
   "configuration": "{\"stateId\":\"<userDefinedState id>\",\"state\":\"ACTIVE\"}"}
  ```
  `state` is `ACTIVE`/`INACTIVE`.
- `SmartPlugOnOffAction` - `{"smartPlugId": "<device id>", "action": "TURN_ON"/"TURN_OFF"}`.
- `RoomClimateAction` - sets a room's heating/cooling target; observed
  fields: `rccId`, `roomControlMode`, `operationMode`,
  `operationModeToggleActive`, `temperature`, `temperatureToggleActive`,
  `scheduleId`, `scheduleName`, `scheduleToggleActive`, `enableBoostMode`,
  `boostModeToggleActive` (the `*ToggleActive` flags gate whether that
  paired field is actually applied by this action, letting one action only
  touch a subset of a room's climate settings).
- `OutdoorCameraAction` - controls a camera's recording/notification/light
  state; observed fields: `cameraId`, `actionEnableCamera` (`TURN_ON`/
  `TURN_OFF`), `actionEnableCameraNotification`
  (`TURN_NOTIFICATION_ON`/...), `lightOptionActive`,
  `actionEnableCameraLight` (`TURN_LIGHT_ON`/`TURN_LIGHT_OFF`),
  `enableCameraAndNotificationOptionActive`.
