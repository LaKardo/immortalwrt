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
  keyed by the seed, build system, tools and toolchain sources. A matching
  cache skips the tools and toolchain build entirely.
- ccache: compiler output, checked by compiler content rather than timestamps.

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
