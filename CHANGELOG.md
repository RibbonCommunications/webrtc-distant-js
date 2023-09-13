# Changelog

- This project adheres to [Semantic Versioning](http://semver.org/).
- This change log follows [keepachangelog.com](http://keepachangelog.com/) recommendations.

## 1.2.0 - 2023-09-13
### Added
- `distant-remote`
  - MacOS: "requestDeviceAuth" method added to remote controller object will check microphone and camera permissions and request them from the user if they've never been prompted or if they were previously denied. The results will be emitted as a "deviceAuth" event which can be listened to using the remote controller's "on" method. `KAJ-1638`
  - "openBrowser" method added to the remote controller object takes an argument of a URL as a string, launches the user's system web browser and opens that URL. `KAJ-1644`

### Changed
- First release of DistantJS published in NPM
- Branding change from @distant to @rbbn
- `distant-vchannel`
  - Bump protobufjs to 7.2.5

## 1.1.1 - 2022-07-21
### Fixed
- `vchannel`
  - Install script no longer throws uncaught exception if command line arguments aren't available as the "npm_config_argv" environment variable. This environment variable was removed in NPM 7, which was released with Node 15. `KAJ-1491`

## 1.1.0 - 2022-07-28
### Added
- `vchannel`
  - getInfo's returned object now contains additional variables to identify os. `KAJ-642`

### Changed
- `vchannel`
  - Bump protobufjs from 6.8.9 to 6.11.3.
- Removed "Update Session" from documentation and from Distant JS. `KAJ-1248`
- Browser errors are now caught when starting a session. `KAJ-1273`

## 1.0.0

### Added

- `vchannel`
  - "cancelWaitForEvents" is an added API function which returns undefined, flushes all calls to "waitForEvents" and causes the pending promises associated with those calls to be rejected

### Changed
- `distant-controller`
  - Calling `close()` on the a Session object will only cause a `stop` message to be sent, and will no longer close the associated emitter or set the associated channel to undefined

## 0.9.0-beta
