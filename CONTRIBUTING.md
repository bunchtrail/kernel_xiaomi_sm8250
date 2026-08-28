# Contributing

This is a focused kernel fork for Xiaomi `lmi` (POCO F2 Pro / Redmi K30 Pro)
on Android 16. Reports from other SM8250 devices can provide useful evidence,
but do not establish support without a maintainer-approved test scope.

## Reporting a battery problem

Use the **Bug report** issue form and provide enough detail to reproduce and
distinguish the problem:

- exact device model and codename;
- ROM name, version, build date, and Android version;
- output of `uname -a` and the complete kernel release;
- whether the kernel ZIP or boot image was modified after release;
- KernelSU/Magisk state and relevant modules;
- symptoms, timestamps, and reproducible steps;
- battery SOC and voltage/current values around the event;
- charging state and what changed when a charger was connected;
- relevant `dmesg` excerpts if access is available.

Useful read-only data can be collected with:

```sh
uname -a
for property in capacity voltage_now current_now status charge_counter \
  charge_full charge_full_design cycle_count; do
  path="/sys/class/power_supply/battery/$property"
  [ -r "$path" ] && printf '%s=%s\n' "$property" "$(cat "$path")"
done
```

Not every property is available on every ROM. Do not bypass device security
solely to collect a report.

Remove account names, serial numbers, phone numbers, network identifiers,
tokens, keys, and unrelated log content before uploading. Do not attach random
vendor blobs, partition dumps, private `boot.img`/`dtbo.img` backups, or other
copyrighted binaries.

## Compatibility reports

Use the **Compatibility report** form after testing a published build on
`lmi`. Report recovery, boot, KernelSU, radio, Wi-Fi, Bluetooth, camera,
charging, and battery behavior separately. A boot-success report is useful but
does not validate the low-battery fix.

## Kernel pull requests

Keep changes reviewable and narrowly scoped:

1. Explain the observed problem and why the change belongs in this tree.
2. Link the original or upstream source when backporting code.
3. Use small, logically separated commits with clear subjects.
4. State the exact base and build result.
5. State the test device, ROM, and runtime result when applicable.
6. Describe failure modes, compatibility risks, and rollback considerations.
7. Preserve existing authorship, SPDX identifiers, and license notices.

Do not combine kernel behavior changes with project branding or unrelated
formatting. Build-tested and hardware-tested are different claims; label them
accurately.
