
# Change Log
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/)
and this project adheres to [Semantic Versioning](http://semver.org/).

## [3.19.0] - 2025-07-31
### Changed
- New: SHC Info
- New: Relay
- New: Multiroom Boiler Control
- Improvements: API extensions

## [3.12.0] - 2024-07-31
### Changed
- New: Automation trigger
- Improvements: OpenAPI documentation (thanks @SteffenMangold)
  
## [3.8.0] - 2023-12-05
### Changed
- New: UserDefinedStates

## [3.2.1] - 2023-02-09
### Changed
- Added OpenAPI documentation

## [3.2.0] - 2022-12-09
### Changed
- Updated HTTP request header to use "api-version : 3.2" instead of "api-version : 2.1"
- Synced Postman collection version with API version 3.2
- Updated Postman environment variables
- Removed deprecated requests for Intrusion Detection from Postman collection
- New: Door/Window Contact II Bypass and Vibration states
- New: Motion Detector Illuminance state (in Lux)
- New: Shutter Control II BlindsControl state for controlling the angle of blinds
- New: Shutter Control II BlindsSceneControl state for controlling the angle and level of blinds at the same time
- Hint: Light / Shutter Control II works with child devices using "#" in the device IDs; make sure to escape "#" in URLs with "%23"
- Hint: Light Control II can control two lamps (two child devices); PowerSwitch state must be PUT/GET on child devices
- Hint: PowerMeter state also works with Light / Shutter Control II now

## [2.1.0] - 2021-01-22
### Changed
- Updated HTTP request header to use "api-version : 2.1" instead of "api-version : 1.0"
- Synced Postman collection version with API version 2.1
- Updated Postman environment variables
- Added new requests for Intrusion Detection in Postman collection in folder Domains
- Deprecated old requests for Intrusion Detection (will be removed in the future) in Postman collection

### Deprecated
- API 1.x will no longer be supported beginning with the Smart Home Controller update scheduled for May 2021

## [0.5.0] - 2021-01-20
### Changed
- Initial release
