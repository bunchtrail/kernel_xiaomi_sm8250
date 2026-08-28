# Source audit

Audit date: 2026-08-28  
Branch: `rapid-soc-dec-fix`  
Audited head: `e938221dd41ac0882983beb139ead5b4a06f8a1e`  
Upstream base: `v311c_A16_KSUN` / `eea388a42674f841b42700152bebab9efec95c1e`

## Result

The branch is five commits ahead of the stated base and zero commits behind
that base. The remote comparison contains four changed paths:

<!-- markdownlint-disable MD013 -->

| Path | Change | Purpose |
| --- | ---: | --- |
| `drivers/power/supply/qcom/qpnp-fg-gen4.c` | +18 / -0 | 17-line upstream fix plus one required forward declaration |
| `arch/arm64/configs/lmi_defconfig` | +1 / -1 | Add `_fix1` to `CONFIG_LOCALVERSION` |
| `.github/workflows/build-rapidfix.yml` | +106 / -0 | Reproducible CI build and packaging |
| `patches/rapid_soc_dec_fix_v311c.patch` | +50 / -0 | Reference copy of the driver patch |

<!-- markdownlint-enable MD013 -->

Only the driver and defconfig alter kernel source or configuration. No other
kernel-source path differs from the stated base.

## Driver verification

The functional code matches Yanfeng Lee's original 17-line change from
[`83cb4c684d0a483e8c2c39f6ae80be428b855d25`](https://github.com/liyafe1997/Xiaomi-fix-battery-one-percent/commit/83cb4c684d0a483e8c2c39f6ae80be428b855d25):

- test filtered `vbatt_avg > 3700` mV;
- require device-tree and runtime `rapid_soc_dec` flags;
- call `fg_gen4_rapid_soc_config(chip, false)`;
- report a configuration error and clear the runtime flag;
- retain the upstream `vbatt_low` reset branch when the DT option is disabled.

The v311c tree defines `fg_gen4_rapid_soc_config()` after its new call site, so
the backport adds this declaration near the existing static declarations:

```c
static int fg_gen4_rapid_soc_config(struct fg_gen4_chip *chip, bool en);
```

## Configuration verification

`arch/arm64/configs/lmi_defconfig` changes only:

```diff
-CONFIG_LOCALVERSION=-Perf_LMI_v311c_A16__KSUN_raystef66
+CONFIG_LOCALVERSION=-Perf_LMI_v311c_A16__KSUN_raystef66_fix1
```

## Build verification

GitHub Actions run
[`33158764783`](https://github.com/bunchtrail/kernel_xiaomi_sm8250/actions/runs/33158764783)
completed successfully at head `e938221dd41ac0882983beb139ead5b4a06f8a1e`.
It fetched and checked clang `r563880c`, checked out KernelSU-Next at
`99155969e37457a8ca0fa6c17c788cc2a0b67f0c`, built `Image`, packaged
AnyKernel3, and generated a checksum.

The downloaded ZIP was read completely without archive errors. Its SHA-256 is:

```text
f1d1c9535ce8bd66434659151a112f1c383b1c4cef95b6369faf87e6ac1f2909
```

This proves source/build/package integrity for the audited commit. Runtime
behavior remains unverified until the user performs the hardware test.

## Local checkout note

The kernel tree contains paths that collide on ordinary case-insensitive
Windows filesystems (for example `aux.c`/`AUX.c` families). A Windows checkout
can therefore show apparent deletions that are not present in the remote
branch. This audit uses GitHub's remote commit comparison for the authoritative
path list.
