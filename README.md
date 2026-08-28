# Perf LMI v311c A16 — `rapid_soc_dec` fix

[![Build](https://github.com/bunchtrail/kernel_xiaomi_sm8250/actions/workflows/build-rapidfix.yml/badge.svg?branch=rapid-soc-dec-fix)](https://github.com/bunchtrail/kernel_xiaomi_sm8250/actions/workflows/build-rapidfix.yml)

This repository maintains an Android 16 kernel build for Xiaomi `lmi`
(POCO F2 Pro / Redmi K30 Pro). It is based on
[raystef66's Perf kernel](https://github.com/raystef66/kernel_xiaomi_sm8250),
tag [`v311c_A16_KSUN`](https://github.com/raystef66/kernel_xiaomi_sm8250/releases/tag/v311c_A16_KSUN),
and backports a runtime exit for the Qualcomm PM8150 FG Gen4
`rapid_soc_dec` path associated with the battery-stuck-at-0–1% symptom.

The kernel builds successfully in GitHub Actions. Validation of the battery
fix on physical hardware is still pending; a successful build does not prove
the runtime behavior.

## Project overview

<!-- markdownlint-disable MD013 -->

| Item | Value |
| --- | --- |
| Devices | POCO F2 Pro / Redmi K30 Pro |
| Codename | `lmi` |
| Android | Android 16; primarily intended for testing on crDroid 12.x |
| Upstream | `raystef66/kernel_xiaomi_sm8250` |
| Base | tag `v311c_A16_KSUN`, commit `eea388a42674f841b42700152bebab9efec95c1e` |
| Kernel release | `4.19.311-Perf_LMI_v311c_A16__KSUN_raystef66_fix1` |
| KernelSU-Next | `99155969e37457a8ca0fa6c17c788cc2a0b67f0c` |
| Toolchain | AOSP clang `r563880c`, clang 21.0.0, build 14054515 |

<!-- markdownlint-enable MD013 -->

This fork targets `lmi` only. Support is not claimed for other SM8250
devices without device-specific testing.

## Why this fork exists

On the affected test device, the reported battery state of charge (SOC) has
remained at 0–1% for hours despite usable charge remaining. Two observations
that motivated the investigation were:

- On 30 July, the displayed SOC stayed around 1%→0% for roughly seven hours,
  then jumped from about 0% to 97% after a charger was connected.
- On 5 August, the device remained at 0% for an extended period, then jumped
  from about 0% to 32% after charging began.

These are observations from one device, not expected values or proof of
behavior on every `lmi`.

## Root cause

The PM8150 FG Gen4 driver has a `rapid_soc_dec` mode. When the low-battery IRQ
path observes voltage below the configured cutoff and the device-tree feature
is enabled, the driver can configure a maximum cutoff current and slope-limit
coefficient so SOC decreases rapidly. The tested device's live device tree
contains `qcom,rapid-soc-dec-en` at:

```text
/proc/device-tree/soc/qcom,spmi@c440000/qcom,pm8150b@2/qpnp,fg/qcom,rapid-soc-dec-en
```

The source tree had paths that enter this mode and revert it during driver
shutdown, but no equivalent runtime exit when average battery voltage later
recovered. Separately, Xiaomi's `shutdown_delay` capacity handling can keep the
reported capacity at 1% under its voltage and charging conditions. These
source-proven behaviors are consistent with the observed symptom, but the
causal fix still requires hardware validation.

See [the technical analysis](docs/rapid-soc-dec.md) for the evidence boundaries
and driver paths.

## The fix

The backport checks the filtered average battery voltage while SOC scaling is
evaluated. If `rapid_soc_dec` is active and `vbatt_avg` rises above 3700 mV, it
calls:

```c
fg_gen4_rapid_soc_config(chip, false);
```

This restores the normal slope-limit coefficient and configured cutoff
current, then clears the runtime state flag. The branch also adds a forward
declaration required by the function order in the v311c driver.

The original fix is by Yanfeng Lee (`liyafe1997`):
[commit `83cb4c684d0a483e8c2c39f6ae80be428b855d25`](https://github.com/liyafe1997/Xiaomi-fix-battery-one-percent/commit/83cb4c684d0a483e8c2c39f6ae80be428b855d25).
This repository adapts that work to raystef66's Android 16 v311c tree; it does
not claim authorship of the original idea.

## Changes from upstream

- Backport the runtime exit from `rapid_soc_dec` in
  `drivers/power/supply/qcom/qpnp-fg-gen4.c`.
- Add `_fix1` to `CONFIG_LOCALVERSION` in `lmi_defconfig`.
- Add a reproducible GitHub Actions build and checksum-producing packaging.
- No other intentional kernel-source changes.

The exact comparison is recorded in [the source audit](docs/source-audit.md).

## Build

The `Build Perf LMI v311c (rapid_soc_dec fix)` workflow is manually dispatched
from the **Actions** tab. It fetches AOSP clang `r563880c` through Git LFS,
checks the reported clang revision, checks out KernelSU-Next at the pinned
commit, verifies the source markers, builds `lmi_defconfig`, validates `Image`,
and packages an AnyKernel3 ZIP with `sha256.txt`.

For the exact commands and environment, see [BUILDING.md](BUILDING.md). The
workflow is the source of truth.

## Installation

Release assets are AnyKernel3 flashable ZIPs, not raw boot images. General
installation outline:

1. Confirm that the bootloader is unlocked and a compatible recovery is
   available.
2. Back up the device's current `boot.img` and `dtbo.img`; keep those backups
   private and off the public repository.
3. Verify the downloaded ZIP against the published SHA-256 checksum.
4. Install the AnyKernel3 ZIP through recovery.
5. Reboot and verify the kernel release with `uname -r`.

Do not pass the ZIP to `fastboot flash boot`. Flashing a custom kernel can make
the device unbootable or cause data loss; the user accepts that risk.

## Verification

After booting, check the kernel release and root integration:

```sh
uname -r
su -c id
```

`uname -r` should report:

```text
4.19.311-Perf_LMI_v311c_A16__KSUN_raystef66_fix1
```

Useful read-only battery values are available with:

```sh
for property in capacity voltage_now current_now charge_full \
  charge_full_design cycle_count; do
  printf '%s=' $property
  cat /sys/class/power_supply/battery/$property
done
```

Follow [the runtime validation plan](docs/runtime-validation.md) before making
claims about the fix.

## Rollback

Keep known-good `boot.img` and `dtbo.img` backups before installation. Restore
the known-good boot image using the device's compatible recovery or documented
fastboot procedure. Restore `dtbo.img` only if it was changed. Exact partition
layout and recovery steps are device-specific; verify them before writing any
partition.

User backup images must never be committed or uploaded as release assets.

## Known limitations

- Runtime battery behavior has not yet been validated with this build on
  physical hardware.
- CI proves that the source compiles and packages; it does not prove the
  low-battery symptom is resolved.
- The project targets Xiaomi `lmi` only.
- Compatibility with other SM8250 devices is not claimed.
- WSL2 has not been validated as a build environment for this tree.

## Contributing and security

Use the issue forms for reproducible bug and compatibility reports. See
[CONTRIBUTING.md](CONTRIBUTING.md) before submitting logs or a pull request.
For sensitive reports, follow [SECURITY.md](SECURITY.md).

## Licensing

This is a Linux kernel source fork. The tree retains its upstream `COPYING`,
SPDX identifiers, license texts, and notices. The top-level
[`COPYING`](COPYING) states `GPL-2.0 WITH Linux-syscall-note` for the kernel and
notes that other licenses can apply to individual files. No separate project
license is substituted for those terms.

## Credits

- [raystef66](https://github.com/raystef66) for the Perf LMI kernel and v311c
  Android 16 base.
- [Yanfeng Lee / liyafe1997](https://github.com/liyafe1997/Xiaomi-fix-battery-one-percent)
  for the original `rapid_soc_dec` runtime-exit fix.
- Xiaomi and Qualcomm for the applicable kernel source components and notices.
- [KernelSU-Next](https://github.com/rifsxd/KernelSU-Next) and its contributors.
- [AnyKernel3](https://github.com/osm0sis/AnyKernel3) by osm0sis and its
  contributors for the packaging framework.
- crDroid is named only as the test ROM environment; this project is not
  affiliated with or endorsed by crDroid.
