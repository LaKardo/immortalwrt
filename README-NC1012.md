# Netcraze NC-1012 and NC-1812 — ImmortalWrt 25.12.2

This branch backports official upstream support for the Netcraze NC-1012 and
NC-1812 to the ImmortalWrt `v25.12.2` tree. Builds are produced by GitHub
Actions.

## Build

Use **Actions → Build Netcraze NC-1012 and NC-1812 → Run workflow**, or push a
tag matching `v25.12.2-netcraze.*`. A tag creates a GitHub Release containing
the images of both devices and the configuration used for the build.

Both devices are built from `configs/netcraze.seed` in one multi-profile build
with per-device root filesystems: the toolchain, kernel and shared packages are
compiled once, and each image contains only the packages of its own device.

Tag pushes dispatch the build on `nc1012-25.12`, checking out the exact tag.
This shares the branch-scoped GitHub Actions cache between releases. The fork's
default branch is `nc1012-25.12` so manual dispatch is also available.

Caches:

- `dl`: source downloads, keyed by the package, tool and feed definitions.
- host tools and toolchain: `staging_dir/host` and `staging_dir/toolchain-*`,
  keyed by the build system, tools and toolchain sources and by
  `toolchain-config.txt`. A matching cache skips the tools and toolchain build
  entirely.
- ccache: compiler output, checked by compiler content rather than timestamps.

`toolchain-config.txt` is the resolved `.config` after `make defconfig` without
the package selection (`CONFIG_PACKAGE_*`, `CONFIG_MODULE_*`,
`CONFIG_TARGET_DEVICE_PACKAGES_*`, `CONFIG_DEFAULT_*`). Adding or removing
packages in the seed therefore keeps the cached toolchain, while any other
option (GCC version, libc, optimisation, target, `CONFIG_USE_*`) changes the key
and rebuilds it. The restored stamps are refreshed, so OpenWrt itself does not
detect a stale toolchain: the key is the only guard. If the toolchain must be
rebuilt for a reason the key does not cover, bump `netcraze-toolchain-v1` in the
workflow. The file is part of the build artifacts.

Caches are saved only after the corresponding step did real work, so an early
failure cannot replace a populated cache with an empty one. Superseded caches
are removed after a newer one is saved. Compiler cache statistics appear in the
job summary.

The workflow also applies the FT roaming fixes from OpenWrt PR #23181
(mac80211, adapted to backports 6.18.39) and the companion hostapd patch, and
verifies that both patches apply before the main build starts.

Each device produces `squashfs-factory.bin`, `squashfs-sysupgrade.bin`, and
the initramfs kernel image. Only flash the images whose name matches the
device. The normal package feeds remain those of ImmortalWrt 25.12.2; kernel
modules must match the resulting kernel ABI.

## Built-in kernel modules

The kernel hash of this build differs from the official ImmortalWrt 25.12.2
release, so `kmod-*` packages from the official feeds cannot be installed on
it. Kernel modules needed later must be added to `configs/netcraze.seed`. Both
images include:

- `kmod-amneziawg` (AmneziaWG 3.1.20260906 from the `awg` feed,
  [awg-openwrt](https://github.com/Slava-Shchipunov/awg-openwrt) pinned to the
  same commit as [Truba](https://github.com/LaKardo/truba)), with its
  dependencies `kmod-udptunnel4`, `kmod-udptunnel6`,
  `kmod-crypto-lib-chacha20poly1305` and `kmod-crypto-lib-curve25519`;
- `kmod-nft-socket`, required by `truba`.

The workflow checks that these modules and packages are present in both
sysupgrade images and records the kernel version of each image in
`kernel-versions.txt`.

Truba and its user-space packages do not depend on the kernel hash and are
installed from the Truba feed after flashing:

```sh
wget -O /etc/apk/keys/truba.pem https://lakardo.github.io/truba/keys/truba.pem
. /etc/os-release; echo "https://lakardo.github.io/truba/${VERSION}/mediatek/filogic/packages.adb" > /etc/apk/repositories.d/truba.list
apk update && apk add amneziawg-tools luci-proto-amneziawg truba luci-app-truba luci-i18n-truba-ru luci-app-upnp
/etc/init.d/network restart
```

## First installation (NC-1012)

1. Set the TFTP host to `192.168.1.2` and connect it to a LAN port.
2. Rename the factory image to `NC-1012_recovery.bin`.
3. Power off the router, hold Reset, power it on, and keep Reset held for
   approximately 10 seconds.
4. Wait for the recovery write and reboot. The router should be available at
   `192.168.1.1`.

The stock recovery image can be restored using the same procedure.

The SFP side of the combo port is supported. The copper EN8811H side may
require the later phylink changes that are not part of the 25.12 kernel tree.

## Upgrading

On a router already running ImmortalWrt, install the matching
`squashfs-sysupgrade.bin`. Packages installed manually after the previous
installation are not part of the image and must be reinstalled.
