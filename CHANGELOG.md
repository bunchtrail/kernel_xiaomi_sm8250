# Changelog

All notable fork-specific changes are documented here. This changelog does not
duplicate the complete upstream kernel history.

## [v311c-A16-fix1] - 2026-08-28

### Added

- Backported Yanfeng Lee's PM8150 FG Gen4 `rapid_soc_dec` runtime exit when
  filtered average battery voltage recovers above 3700 mV.
- Added the forward declaration required by the v311c driver layout.
- Added a reproducible GitHub Actions build using AOSP clang `r563880c` and
  KernelSU-Next commit `99155969e37457a8ca0fa6c17c788cc2a0b67f0c`.
- Added AnyKernel3 packaging and SHA-256 generation.

### Changed

- Appended `_fix1` to `CONFIG_LOCALVERSION`.

### Validation

- Build and package verified in GitHub Actions.
- Runtime validation on physical hardware is pending.
- No other intentional kernel-source changes from `v311c_A16_KSUN`.

[v311c-A16-fix1]: https://github.com/bunchtrail/kernel_xiaomi_sm8250/releases/tag/v311c-A16-fix1
