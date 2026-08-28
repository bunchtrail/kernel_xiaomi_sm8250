# Pull request

## Problem

Describe the user-visible or source-level problem and its scope.

## Source or upstream reference

Link the original patch, upstream commit, specification, or other evidence.
Preserve authorship when adapting existing work.

## Changes

List the functional changes and explain why each one is necessary.

## Build status

- Base commit:
- Defconfig:
- Toolchain:
- Build result or CI URL:

## Runtime testing

- Test device and codename:
- ROM and Android version:
- Kernel release (`uname -r`):
- Tested features:
- Result:

Use ?not tested? where applicable. Do not imply that a successful build is a
hardware validation.

## Risks and rollback

Describe likely regressions, affected paths, failure behavior, and how the
tester can return to a known-good kernel.

## Checklist

- [ ] The commits are small and logically separated.
- [ ] Existing SPDX identifiers, authorship, and license notices are preserved.
- [ ] No secrets, personal data, device backups, or unrelated binary blobs are included.
- [ ] The build status is stated accurately.
- [ ] Hardware test claims identify the actual device and result.
