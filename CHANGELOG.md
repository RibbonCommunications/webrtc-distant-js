Kandy Distant JS change log.

- This project adheres to [Semantic Versioning](http://semver.org/).
- This change log follows [keepachangelog.com](http://keepachangelog.com/) recommendations.


## 1.1.0

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
