# Codex for Open Source application draft

Internal maintainer draft. Do not submit without reviewing every field and
explicitly authorizing submission.

Official form: <https://openai.com/form/codex-for-oss/>  
Criteria checked: 2026-08-28

The current form asks for a public GitHub profile and repository, maintainer
role, why the repository qualifies (500 characters), requested program
benefits, OpenAI organization ID, intended API-credit use (500 characters),
and an optional final note (500 characters). OpenAI states that maintainers of
active open-source projects may apply and considers meaningful usage, broad
adoption or clear ecosystem importance, plus evidence of active maintenance.

## Project

`bunchtrail/kernel_xiaomi_sm8250` maintains an Android 16 kernel for Xiaomi
`lmi` (POCO F2 Pro / Redmi K30 Pro). It preserves a focused backport for a
Qualcomm PM8150 FG Gen4 `rapid_soc_dec` battery-reporting issue on top of
raystef66's Perf v311c kernel, with pinned inputs, a reproducible build,
checksummed AnyKernel3 releases, runtime-test guidance, and upstream
attribution.

## Maintainer work

Current and expected maintenance includes:

- investigating kernel and fuel-gauge source paths;
- adapting and reviewing narrow backports;
- maintaining pinned CI and release packaging;
- reviewing build failures and regression reports;
- triaging battery and compatibility issues;
- analyzing redacted device logs;
- documenting installation, rollback, and validation;
- submitting minimal fixes upstream when appropriate.

## Ecosystem importance

The repository is new and does not yet demonstrate broad adoption. Its honest
case is niche importance: preserving modern Android 16 kernel support for an
older Snapdragon 865 device family, documenting a kernel-level battery symptom
and its prior fix, and making a previously manual build reproducible and
auditable. This is maintenance of inherited Linux/Android kernel work, not a
claim that the fork authored the whole kernel.

## Usage snapshot

Snapshot at 2026-08-28 before the first fork-specific release:

- GitHub stars: 0
- Direct forks of this fork: 0
- Open issues: 0 (issues were not yet enabled)
- Fork-specific release downloads: 0 (no release existed yet)
- External users: not established
- Fork-specific contributors: one maintainer; the inherited history contains
  many upstream authors and must not be counted as fork contributors

Update these numbers from GitHub immediately before applying. Do not substitute
the upstream repository's release downloads or network size for this fork's
usage.

## Why Codex helps

Codex has already assisted with source comparison, backport preparation,
workflow debugging, exact toolchain retrieval, artifact verification, and OSS
documentation. Continued use would support kernel-source investigation,
regression analysis, review of small backports, CI maintenance, release
automation, log triage, issue review, and documentation while keeping final
maintainer decisions and device flashing under human control.

## Form draft

### Identity fields

- First name: **maintainer must provide**
- Last name: **maintainer must provide**
- Email associated with ChatGPT: **maintainer must provide**
- GitHub username: `bunchtrail` (confirm the profile is public)
- Repository URL: `https://github.com/bunchtrail/kernel_xiaomi_sm8250`
- Role: **Primary maintainer** (confirm before submission)
- Interested in: API credits; consider Codex Security only if the maintainer
  intends to operate the required security workflow
- OpenAI organization ID: **maintainer must provide**

### Why does this repository qualify? (maximum 500 characters)

<!-- markdownlint-disable-next-line MD013 -->
> I maintain an Android 16 kernel fork for Xiaomi lmi (POCO F2 Pro / Redmi K30 Pro), preserving a targeted PM8150 FG Gen4 battery-reporting fix with reproducible builds, checksummed releases, rollback guidance, issue triage, and upstream attribution. The repository is new and has limited adoption, but it supports continued use of a Snapdragon 865 device and turns a fragile manual kernel workflow into maintainable OSS infrastructure.

### How will you use API credits? (maximum 500 characters)

<!-- markdownlint-disable-next-line MD013 -->
> I would use API credits for maintainer automation: comparing kernel branches, reviewing small backports, classifying CI failures, summarizing redacted device logs, checking release artifacts and documentation, drafting issue responses, and preparing minimal upstream contributions. Human review would remain required for merges, releases, security decisions, and all device flashing or runtime validation.

### Anything else we should know? (maximum 500 characters)

<!-- markdownlint-disable-next-line MD013 -->
> Codex already helped trace the fuel-gauge behavior, adapt the original fix, debug the pinned Android clang build, and verify the first artifact. Build validation is complete; low-battery hardware validation is still pending and is described honestly. The project credits raystef66, Yanfeng Lee, KernelSU-Next, AnyKernel3, Xiaomi, and Qualcomm, and does not claim ownership of inherited kernel work.

## Weaknesses to disclose

- 0 stars and 0 direct forks at the current snapshot.
- No demonstrated external users or fork-specific download history yet.
- No issue-triage or review history yet because the repository was created
  today and issues were disabled.
- Only one current maintainer.
- The runtime fix has not been validated on hardware.
- The upstream project has no visible pull-request history and the fork has not
  yet established accepted upstream activity.
- The repository's case is niche maintenance value, not broad adoption.

These weaknesses may make the application premature. A stronger application
would include organic usage, real issue/review history, a published release,
hardware validation, and an upstream response. Do not manufacture those
signals.
