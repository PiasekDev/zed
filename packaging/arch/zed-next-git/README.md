# zed-next-git

This is the clean source-build package for the `next` branch in
`PiasekDev/zed`. It clones the branch into makepkg's source tree, builds a
release Linux bundle, then installs that bundle as a replacement for Arch's
`zed` package.

Build and install:

```sh
cd packaging/arch/zed-next-git
makepkg -Csi
```

Disable `target-cpu=native`:

```sh
ZED_NEXT_TARGET_CPU= makepkg -Csi
```

For day-to-day iteration from an existing checkout, `script/package-zed-next-local`
is usually faster because it reuses this repository's local `target` directory.
