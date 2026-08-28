# Runtime validation plan

This procedure is for the device owner after choosing to install the release.
It does not authorize automated flashing. The current project status is
**build verified; runtime fix validation pending**.

## Before installation

1. Confirm the device is Xiaomi `lmi` (POCO F2 Pro / Redmi K30 Pro), uses the
   expected Android 16 ROM, has an unlocked bootloader, and has a compatible
   recovery.
2. Keep known-good private backups of `boot.img` and `dtbo.img` outside the
   repository and phone storage where possible.
3. Confirm a tested recovery or fastboot rollback procedure before changing
   the boot partition.
4. Compare the downloaded ZIP with the release `sha256.txt`.
5. Record the current `uname -r`, root state, ROM build, and basic device
   behavior.

Do not upload backup images or raw device identifiers to a public issue.

## Initial boot validation

After installing the AnyKernel3 ZIP through recovery and rebooting, verify:

```sh
uname -r
su -c id
```

Expected kernel release:

```text
4.19.311-Perf_LMI_v311c_A16__KSUN_raystef66_fix1
```

If the release string is different, stop the battery test and determine which
kernel actually booted.

Check each item independently:

- normal boot and unlock;
- mobile network, calls, and data as normally used;
- Wi-Fi;
- Bluetooth;
- camera;
- charging and charger detection;
- KernelSU/root behavior expected by the owner;
- basic performance, temperature, sleep, and wake behavior.

If a critical regression appears, stop testing and restore the known-good boot
image using the pre-verified rollback procedure.

## Baseline battery sample

Record a small baseline before the low-battery phase:

```sh
date -Is
for property in capacity voltage_now current_now status charge_counter \
  charge_full charge_full_design cycle_count; do
  path="/sys/class/power_supply/battery/$property"
  [ -r "$path" ] && printf '%s=%s\n' "$property" "$(cat "$path")"
done
```

Values normally use kernel power-supply class units; preserve the raw values
and interpret units separately rather than silently converting them.

## Natural low-battery test

Use the phone normally and observe one natural low-charge cycle. Do not force
multiple deep discharges and do not leave an unstable device unattended.

Near low charge, take timestamped samples periodically and whenever behavior
changes. Record at least:

- timestamp;
- `capacity`;
- `voltage_now`;
- `current_now`;
- `status`;
- `charge_counter`, if exposed;
- whether the screen and a charger are connected;
- any reboot, shutdown, prolonged 0?1% display, or implausible SOC jump.

A simple manual sample command is:

```sh
printf '\n--- %s ---\n' "$(date -Is)"
for property in capacity voltage_now current_now status charge_counter; do
  path="/sys/class/power_supply/battery/$property"
  [ -r "$path" ] && printf '%s=%s\n' "$property" "$(cat "$path")"
done
```

Connect a charger when it is naturally appropriate and continue sampling long
enough to detect an immediate SOC jump. Device safety and battery health take
priority over extending the test.

## Result classification

- **Basic compatibility passed:** the expected kernel booted and core device
  functions showed no regression during ordinary use.
- **Battery behavior encouraging:** the prior symptom did not occur during one
  natural cycle. This is useful evidence, not universal proof.
- **Fix reproduced:** the prolonged 0?1% behavior or implausible charger-time
  jump still occurred with timestamped data.
- **Inconclusive:** the kernel identity, test conditions, or logs are
  insufficient to compare behavior.

Report the exact result without changing ?runtime validation pending? to
?verified? unless the test actually exercised the relevant low-battery path.

## Public report hygiene

Before sharing, remove names, serial numbers, phone numbers, network
identifiers, account data, tokens, unrelated logs, and filesystem paths that
reveal personal information. Do not share `boot.img`, `dtbo.img`, or other
partition images.
