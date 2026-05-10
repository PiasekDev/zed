# zed-next-local

This package installs a release tarball built from the current checkout. Use it
when you want pacman ownership but also want to reuse this repository's local
`target` directory and native Rust optimization settings.

Build and install from the repository root:

```sh
script/package-zed-next-local
```

Package an existing local bundle without rebuilding:

```sh
script/package-zed-next-local --no-build
```

Disable `target-cpu=native`:

```sh
ZED_NEXT_TARGET_CPU= script/package-zed-next-local
```

The helper temporarily compiles as the stable channel, embeds
`ZED_UPDATE_EXPLANATION` to disable upstream auto-update, restores
`crates/zed/RELEASE_CHANNEL`, and packages the resulting tarball with makepkg.
