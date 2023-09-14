# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## 2.1.0 - 2023-09-13
### Added
- Direct messaging from the Screen Tracker to the remote via the session controller for sharing of display information

### Fixed
- Using Distant Tracker 2.0.0 in Electron 4 would cause an error to be thrown when `@rbbn/distant-tracker/preload/main` was `require`d in Electron's main process (KAJ-1439)

## 2.0.0 - 2022-08-12

### Added
- The installation package now includes two preload scripts. **`preload/main.js` must be `require`d in the Main process even if Electron context isolation is not used**. `preload/renderer.js` must be `require`d in the app's preload script if context isolation is used.

### Changed
#### Window Tracker
- The `createWindowTracker()` API should now receive the ID of the Electron window being tracked, not the window object itself.
- When the windowTracker emits events, event.target will be the ID of the window that moved, and not the window object.

#### Screen Tracker
- The `createScreenTracker()` API now accepts an optional object with a single `rendererLogs` parameter to enable debug logs.
- The `topLeftPointUpdate` event is renamed `displaysUpdated`.
- The `getTopLeftPoint` method is replaced by `normalizeBounds`. This method now performs **all** normalization required to interwork with the Distant VDI platform, meaning the application need not perform any additional normalization.

See README for details on usage on all of the above.

## 1.0.0
Initial release.

<!-- changelog possible fields:
### Added
### Changed
### Removed
### Deprecated
### Fixed
### Security
