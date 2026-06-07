# Changelog

All notable changes to **matterbridge-ecovacs-yeedi** are documented here.

This project loosely follows [Semantic Versioning](https://semver.org/):
patch releases (x.x.N) are fixes, minor releases (x.N.0) add features, and
major releases (N.0.0) introduce breaking changes.

## [1.1.4] - 2026-06-07

### Changed
- The Empty Dust Bin and Clean Completed switches now use the basic On/Off
  cluster instead of the Lighting variant. This removes the spurious "Power-on
  behavior" configuration dropdown that Home Assistant showed for these two
  switches under Matterbridge 3.8.0. The switches' on/off behavior is unchanged.

### Added
- CHANGELOG.md is now included in the published package.

## [1.1.3] - 2026-06-07

### Fixed
- Restored all child endpoints (Filter Life, Main Brush Life, Side Brush Life,
  Empty Dust Bin, Clean Completed) that disappeared after Matterbridge updated
  to 3.8.0 (Matter 1.5.1 / matter.js 0.17.1). Under 3.8.0, child endpoints are
  attached asynchronously; configuring their clusters across separate
  statements caused an "uninitialized-dependency" error that aborted device
  initialization after only the base Run Mode and Battery clusters were added.
- Child endpoints are now created using the chained pattern used by the
  official 3.8.0 devices: cluster creation is chained directly onto
  `addChildDeviceType(...)` and finished with `addRequiredClusterServers()` in a
  single statement, so each child is fully configured before its async attach
  completes.

### Notes
- Verified to build against both Matterbridge 3.8.0 and 3.7.10, so installs on
  either version continue to work.

## [1.1.2] - 2026-06-07

### Fixed
- Matterbridge 3.8.0 compatibility work (republish of the 1.1.1 fix).

## [1.1.1] - 2026-06-07

### Fixed
- First attempt at Matterbridge 3.8.0 compatibility for the disappearing child
  endpoints. Superseded by 1.1.3, which contains the complete, verified fix.

## [1.0.9] - 2026-06-05

### Added
- Full README with features, installation, configuration, room/segment
  cleaning, the Clean Completed automation example, troubleshooting, and
  credits. This is also what now renders on the npm package page.

## [1.0.8] - 2026-06-05

### Added
- "Clean Completed" switch endpoint. Turns ON only after the robot finishes a
  cleaning run and returns to the dock with no error; turns OFF when a new run
  starts. Intended for safe daily-run automations that must not trigger twice.

## [1.0.7] - 2026-05-31

### Fixed
- Error state was being lost inside the state debounce window: a
  `CleanReport('pause')` arriving within ~80ms could overwrite an Error state.
  The state flusher now forces the Error state whenever an active error code is
  set, regardless of a competing clean report.

## [1.0.6] - 2026-05-31

### Fixed
- Root cause of misreported errors: the underlying library emits the error code
  as a string (e.g. '103'), so the strict-equality switch never matched and
  every error defaulted to "Unable to complete operation". Error codes are now
  coerced to a number before mapping.

## [1.0.5] - 2026-05-31

### Fixed
- API-level error codes (>= 10000, e.g. 20003 "wbHeap do not support") are now
  ignored instead of being treated as robot hardware errors and overwriting
  real error codes.

## [1.0.4] - 2026-05-31

### Fixed
- Removed an unreliable Error-string handler and added a safeguard that
  downgrades a stale Error state to Stopped when no active error code is set.

## [1.0.3] - 2026-05-31

### Fixed
- Yeedi map/room loading. Patched the library's map-subset handling to fall
  back to `cid` when `mid` is absent (Yeedi sends `cid`), and added a short
  delayed map fetch after connect so rooms populate reliably.

## [1.0.2] - 2026-05-31

### Changed
- Removed the SetWorkMode call, which the Yeedi Vac Pro does not support.

### Fixed
- Error state propagation and handling of error code 500 (command timeout).

## [1.0.1] - 2026-05-31

### Changed
- Version bump to prevent Matterbridge from auto-reverting the plugin to the
  registry version during development.

## [1.0.0] - 2026-05-31

### Added
- Initial release. Bridges a Yeedi Vac Pro into Matter via Matterbridge, forked
  from `matterbridge-ecovacs` with Yeedi authentication (`authDomain:
  yeedi.com`).
- Battery level, charging state, and voltage.
- Operational state (Docked, Running, Paused, Returning, Charging) and mapped
  operational errors (wheels jammed, brush jammed, navigation sensor obscured,
  low battery, and more).
- Clean mode selector (Vacuum, Mop, Vacuum and Mop, Vacuum then Mop).
- Consumable life sensors for Filter, Main Brush, and Side Brush.
- Empty Dust Bin switch.
- Room/segment cleaning via service call.
