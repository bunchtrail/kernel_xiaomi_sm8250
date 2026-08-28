# Building Perf LMI v311c A16 fix1

The GitHub Actions workflow at
`.github/workflows/build-rapidfix.yml` is the source of truth. Run
[`33158764783`](https://github.com/bunchtrail/kernel_xiaomi_sm8250/actions/runs/33158764783)
is the first successful build of the fix branch.

## Supported build environment

The verified environment is GitHub Actions `ubuntu-latest`. A conventional
Linux host should be able to run the same commands with compatible packages,
but only the Actions environment has been tested for this fork.

Do not use a normal case-insensitive Windows/NTFS checkout. The kernel tree
contains case-sensitive path combinations that Windows cannot represent
faithfully. WSL2 is not yet verified for this project; if attempted, place the
tree on a case-sensitive Linux filesystem rather than a mounted Windows path.

## Inputs pinned by the workflow

<!-- markdownlint-disable MD013 -->

| Input | Value |
| --- | --- |
| Kernel branch | `rapid-soc-dec-fix` |
| Defconfig | `arch/arm64/configs/lmi_defconfig` |
| Architecture | `arm64` |
| AOSP clang directory | `clang-r563880c` |
| clang output | 21.0.0, Android build 14054515 |
| Toolchain repository commit | `b13a969798dd18ef2af4f7225fd271565e7e135f` |
| KernelSU-Next commit | `99155969e37457a8ca0fa6c17c788cc2a0b67f0c` |
| AnyKernel3 base ZIP | upstream `v311c_A16_KSUN` release asset |
| AnyKernel3 base SHA-256 | `a7b9acf5e5d404df5bb88ac0848d5ccd5057b35fe13beb8d09fe6e7c5ea9a3b6` |

<!-- markdownlint-enable MD013 -->

The workflow verifies both the toolchain revision string and the downloaded
AnyKernel3 base digest before building or packaging.

## Dependencies

The verified workflow installs:

```sh
sudo apt-get update
sudo apt-get install -y bc bison flex libssl-dev libncurses5-dev \
  libncurses-dev cpio python3 zip unzip curl wget git build-essential \
  binutils-aarch64-linux-gnu binutils-arm-linux-gnueabi git-lfs
```

## Toolchain

The workflow performs a sparse, no-smudge Git LFS checkout of only
`clang-r563880c` from
`bluestacks/prebuilts-clang-host-linux-x86-a16`, then verifies:

```sh
"$HOME/clang/bin/clang" --version | grep -q "r563880c"
```

The repository commit is pinned in the workflow so a moving branch cannot
silently change the toolchain input.

## KernelSU-Next setup

KernelSU-Next is fetched by exact commit and linked at the location expected by
the kernel tree:

```sh
git init KernelSU-Next
git -C KernelSU-Next remote add origin https://github.com/rifsxd/KernelSU-Next.git
git -C KernelSU-Next fetch --depth 1 origin 99155969e37457a8ca0fa6c17c788cc2a0b67f0c
git -C KernelSU-Next checkout FETCH_HEAD
ln -s ../KernelSU-Next/kernel drivers/kernelsu
test -f drivers/kernelsu/ksu.c
```

Do not update KernelSU-Next independently when reproducing this release.

## Configure and build

From the kernel root after setting up clang and KernelSU-Next:

```sh
export PATH="$HOME/clang/bin:$PATH"
export LD_LIBRARY_PATH="$HOME/clang/lib64:${LD_LIBRARY_PATH:-}"

make O=out ARCH=arm64 lmi_defconfig
make -j"$(nproc)" O=out ARCH=arm64 \
  CC=clang \
  CLANG_TRIPLE=aarch64-linux-gnu- \
  CROSS_COMPILE=aarch64-linux-gnu- \
  CROSS_COMPILE_ARM32=arm-linux-gnueabi- \
  LD=ld.lld \
  AR=llvm-ar NM=llvm-nm OBJCOPY=llvm-objcopy OBJDUMP=llvm-objdump \
  READELF=llvm-readelf OBJSIZE=llvm-size STRIP=llvm-strip
```

Expected kernel output:

```text
out/arch/arm64/boot/Image
```

The workflow fails if `Image` is absent. It also verifies the source fix,
`_fix1` configuration, compiler revision, and generated configuration.

## Packaging

The v311c upstream AnyKernel3 release ZIP is downloaded and verified against
its published GitHub asset digest. The workflow unpacks it, replaces only its
`Image`, repacks:

```text
Perf_LMI_v311c_A16_KSUN_raystef66_fix1.zip
```

and generates:

```sh
sha256sum Perf_LMI_v311c_A16_KSUN_raystef66_fix1.zip | tee sha256.txt
```

Both files are uploaded as one Actions artifact. Packaging does not flash a
device.

## Running in GitHub Actions

1. Open the repository's **Actions** tab.
2. Select **Build Perf LMI v311c (rapid_soc_dec fix)**.
3. Choose **Run workflow** on `rapid-soc-dec-fix`.
4. Wait for all steps to complete.
5. Download the artifact and independently compare the ZIP to `sha256.txt`.
