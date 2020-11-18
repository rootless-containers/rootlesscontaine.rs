---
title: Cgroups
weight: 50
---

## v1

Delegating cgroup v1 controllers to non-root users is not considered to be safe.
So, most Rootless Containers implementations do not support using cgroups on cgroup v1 hosts.

However, LXC supports delegating cgroup v1 to non-root users by using a PAM module called [`pam_cgfs`](https://manpages.debian.org/unstable/lxc/pam_cgfs.8.en.html).

## v2

Unlike cgroup v1, cgroup v2 officially supports delegation.
Most Rootless Containers implementations rely on systemd for delegating v2 controllers to non-root users.
See [Getting Started/Common/Cgroup v2](/getting-started/common/cgroup2/) for the actual configuration.
