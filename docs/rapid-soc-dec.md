# PM8150 FG Gen4 `rapid_soc_dec` analysis

This document separates behavior demonstrated by source code from live-device
observations and from the remaining runtime hypothesis.

## Hardware

The target is Xiaomi `lmi` (POCO F2 Pro / Redmi K30 Pro) using the Qualcomm
PM8150 family fuel gauge and the FG Gen4 driver. This fork is based on the
Android 16 `v311c_A16_KSUN` release from raystef66.

## Device-tree configuration

The driver reads the boolean property `qcom,rapid-soc-dec-en` into
`chip->dt.rapid_soc_dec_en`. The live device tree on the investigated device
contains the property at:

```text
/proc/device-tree/soc/qcom,spmi@c440000/qcom,pm8150b@2/qpnp,fg/qcom,rapid-soc-dec-en
```

That confirms the feature is enabled on that device. It does not, by itself,
prove how often the low-voltage IRQ path is taken.

## Entry path

`fg_vbatt_low_irq_handler()` reads battery voltage when the `VBATT_LOW_IRQ`
fires. When voltage is below `cutoff_volt_mv` and rapid SOC decrease is enabled
by device tree, the handler applies a debounce counter. After the configured
critical-low sequence, it sets `chip->rapid_soc_dec_en = true` and calls:

```c
fg_gen4_rapid_soc_config(chip, true);
```

This entry path is directly established by the source.

## `rapid_soc_dec`

With `en == true`, `fg_gen4_rapid_soc_config()` writes the maximum slope-limit
coefficient and configures the maximum cutoff current. While the runtime flag
is true, the normal slope-limit configuration path returns early and smoothed
battery-capacity overrides are bypassed. With `en == false`, the function
restores `SLOPE_LIMIT_DEFAULT` and the cutoff current from device-tree
configuration.

## Why 1% appears

The driver also contains Xiaomi `shutdown_delay` handling. If reported
capacity reaches 0 while battery voltage remains above the shutdown threshold
and the device is not charging, the capacity property can be reported as 1 and
`shutdown_delay` remains active. This explains a source-level mechanism by
which 1% can continue to be displayed.

The observed long periods at 0–1% and subsequent SOC jumps correlate with
these code paths. Logs proving the exact internal sequence during those
observations were not captured, so correlation is not presented as proof of
causation.

## Missing runtime exit

Before this backport, the tree entered rapid SOC decrease from the low-voltage
IRQ and reverted it during driver shutdown. It did not contain the runtime
voltage-recovery check from Yanfeng Lee's fix. Therefore the runtime flag and
rapid configuration could remain active after filtered average voltage had
recovered, until some other lifecycle event reverted the state.

## Backported fix

`fg_gen4_get_prop_soc_scale()` already reads filtered average battery voltage
and converts it to millivolts. The backport adds a check after that conversion:

1. Require `vbatt_avg > 3700` mV.
2. Require the device-tree feature and active runtime flag.
3. Call `fg_gen4_rapid_soc_config(chip, false)`.
4. Log an error if configuration fails and clear the runtime flag, matching the
   original patch.

The v311c driver also needs a forward declaration because the configuration
function is defined after the new call site.

## Upstream source

The original change is Yanfeng Lee's
[`Add strategy to exit rapid_soc_dec to fix 1% problem`](https://github.com/liyafe1997/Xiaomi-fix-battery-one-percent/commit/83cb4c684d0a483e8c2c39f6ae80be428b855d25)
commit. This repository is a targeted adaptation/backport, not the origin of
the fix.

## Evidence

### Source evidence

- The DT property is parsed by the driver.
- The low-voltage IRQ enables rapid SOC decrease after its debounce path.
- Enabling changes the slope-limit coefficient and cutoff current.
- Capacity smoothing is bypassed while the runtime flag is active.
- `shutdown_delay` can report 1% under its coded conditions.
- The base branch lacked the 3700 mV runtime recovery check.
- The backport matches the original functional change, plus one declaration.

### Live device-tree evidence

The property path above was read on the investigated `lmi` device. This
confirms configuration, not the complete runtime sequence.

### Observed battery behavior

- One low-SOC episode lasted roughly seven hours and was followed by an
  approximately 0%→97% jump when charging began.
- Another extended 0% episode was followed by an approximately 0%→32% jump.

These observations are device-specific. The patched kernel has been
build-tested but not yet subjected to the documented low-battery hardware
validation, so the runtime fix remains pending.
