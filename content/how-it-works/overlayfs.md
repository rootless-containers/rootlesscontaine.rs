---
title: OverlayFS
weight: 30
---

Rootless OverlayFS is supported since [kernel 5.11](https://github.com/torvalds/linux/commit/459c7c565ac36ba09ffbf24231147f408fde4203).

Older kernel releases didn't support rootless OverlayFS,
though [Ubuntu](https://kernel.ubuntu.com/git/ubuntu/ubuntu-bionic.git/commit/fs/overlayfs?id=3b7da90f28fe1ed4b79ef2d994c81efbc58f1144)
supports it by patching the kernel.

[Debian](https://salsa.debian.org/kernel-team/linux/blob/283390e7feb21b47779b48e0c8eb0cc409d2c815/debian/patches/debian/overlayfs-permit-mounts-in-userns.patch)
supports rootless OverlayFS too, when `overlay.ko` is loaded with a custom modprobe option `permit_mounts_in_userns=1`.
However, Debian version of rootless OverlayFS (before kernel 5.11) is [known to be broken](https://github.com/moby/moby/issues/42302) as of April 2021,
while Ubuntu version seems stable.

On other distros, Rootless Containers typically use [fuse-overlayfs](/glossary#fuse-overlayfs) instead of the real OverlayFS.
