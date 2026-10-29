# Bun OS: forward port of the WSL2 patchset to mainline v7.3 (unbuilt)

Textual 3-way port of the delta `v6.18.58 .. linux-msft-wsl-6.18.58.1` onto `v7.3`, plus the Rust config fragment and a build script.
Nothing here has been compiled or booted; the build check is the `wsl-kernel.yml` workflow in aphrody-labs/bun-os.
Details, conflict list and unverified claims: docs/WSL.md in aphrody-labs/bun-os (branch `os`).
