# Aphrody WSL source migration to Linux 7.2

Native base: Aphrody Linux 7.2.9 commit
01fa85c9be7526fe009bedfd280d9c10c903bb91.
Microsoft integration source: owned WSL fork commit
d504d40ab83839cb61942a812a99d183c42227b9, compared with upstream
Linux v6.18.54. Microsoft and upstream notices are retained.

The 44-file Microsoft delta imports 27 added files. Of the 17 modified
files, the version-only Makefile change is excluded, the FUSE file-release
fix already exists upstream, 13 apply directly, and three require context
adaptation: the x86 Hyper-V workaround, Hyper-V Kbuild registration and
SWIOTLB private-pool helper. No graphics UAPI is replaced with a stub.

The wsl profile uses the Microsoft x86_64 or arm64 base configuration,
then applies Aphrody generic flags and explicit Hyper-V/dxgkrnl overrides.
It disables the native NVIDIA HMM profile and Nouveau/Nova. WSL GPU access
uses the Windows host driver; native Linux NVIDIA modules are a separate
native profile and must not be installed in the WSL guest.

## Build and validation

Use a qualified Linux build host. The VPS used Ubuntu 26.04, Clang/LLD 21.1.8,
Rust 1.98.1 and bindgen 0.73.2, with LIBCLANG_PATH=/usr/lib/llvm-21/lib.
From this checkout, set ARCH=x86_64 or ARCH=arm64 and an absolute
KBUILD_OUTPUT, then run scripts/aphrody-config.sh wsl. The profile checks
requested effective configuration values after olddefconfig.

Qualification on 2026-10-10 passed both architecture profiles, a full x86_64
bzImage/modules/headers build, the static bun_accel selftest and kernel/driver
Clippy. Targeted arm64 compilation passed Hyper-V assembly/core, setup, timer,
PCI, DMA, dxgkrnl and the Rust accelerator. A diskless QEMU TCG guest with two
CPUs, 1 GiB RAM and no network booted 7.2.9-aphrody-wsl2+ and passed all seven
bun_accel selftests, including the unprivileged-write rejection. The entire
sequence passed again after whitespace and configuration-file mode fixes.

SHA-256 of the qualified x86_64 artifacts:

- bzImage: 08bf4cd5f319c57f47a8528b00e184ca74bda781935ec1d14e6423424ffa9bea
- bun_accel.ko: 7bbbbf59947a931ff3862154a8c3a607435bb07d5c53f895155fc038c974f42c
- copy_batch: 4493f239267b8590e0fea604a524c4171e6aa56afd8b925a437bab53b9ccbb48

## Style and scope

Strict checkpatch reports 23 errors in the original Microsoft delta and
13 errors in this port. Every remaining error was matched by message,
file and code to the original delta: the one-element arrays in graphics
protocol structures. Their serialized layout is retained. The checkpatch
exit is nonzero; no lint rule is suppressed. Git whitespace checks pass.

The Windows owner workstation stayed native; WSL was not started. These
receipts do not qualify RTX 4070 GPU-PV activation, an arm64 boot, a host
kernel replacement, or memory/performance gains. NVIDIA native-module
qualification belongs to the separate native Linux profile. Follow
https://docs.nvidia.com/cuda/wsl-user-guide/index.html for the WSL CUDA
host-driver/toolkit boundary before any separate activation.
