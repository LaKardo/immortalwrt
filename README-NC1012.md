# Netcraze NC-1012 — ImmortalWrt 25.12.2

This branch backports official upstream support for the Netcraze NC-1012 to
the ImmortalWrt `v25.12.2` tree. Builds are produced by GitHub Actions.

## Build

Use **Actions → Build Netcraze NC-1012 → Run workflow**, or push a tag matching
`v25.12.2-nc1012.*`. A tag creates a GitHub Release containing the images and
configuration used for the build.

The target produces `squashfs-factory.bin`, `squashfs-sysupgrade.bin`, and the
initramfs kernel image. The normal package feeds remain those of ImmortalWrt
25.12.2; kernel modules must match the resulting kernel ABI.

## First installation

1. Set the TFTP host to `192.168.1.2` and connect it to a LAN port.
2. Rename the factory image to `NC-1012_recovery.bin`.
3. Power off the router, hold Reset, power it on, and keep Reset held for
   approximately 10 seconds.
4. Wait for the recovery write and reboot. The router should be available at
   `192.168.1.1`.

The stock recovery image can be restored using the same procedure.

The SFP side of the combo port is supported. The copper EN8811H side may
require the later phylink changes that are not part of the 25.12 kernel tree.
