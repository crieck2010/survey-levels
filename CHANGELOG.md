# Changelog

All notable changes to `survey-levels`. Follows semantic versioning.

## [0.1.0] - 2026-09-23
### Added
- Initial release: pure-Python, dependency-free differential leveling engine.
- `reduction`: height-of-instrument reduction of level runs, sideshot support,
  arithmetic page check.
- `adjustment`: loop misclosure, allowable error (C*sqrt(K) with standard
  first/second/third-order specs), pass/fail check, proportional distribution
  of corrections by setup count.
- 13 unit tests (stdlib `unittest`, no test-runner dependency).
