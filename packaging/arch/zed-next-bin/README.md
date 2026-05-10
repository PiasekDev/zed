# zed-next-bin

This package installs the moving `zed-next` GitHub release from `PiasekDev/zed`
as a replacement for Arch's `zed` package.

Build and install locally:

```sh
cd packaging/arch/zed-next-bin
makepkg -si
```

After a new `zed-next` release is published, rebuild with:

```sh
makepkg -Cfi
```

The package intentionally uses `sha256sums=('SKIP')` because the `zed-next`
release tag is overwritten by the workflow on every push to `next`. For a
public AUR package, switch to immutable release tags and pin the checksum from
the uploaded `zed-next-SHA256SUMS.txt` asset.
