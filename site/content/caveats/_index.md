---
title: Caveats and Future work
weight: 40
---

- The user namespace implementation in the kernel tends to have vulnerabilities.

- Setting up subuid and subgid is sometimes a mess, especially on LDAP/AD environments.
  - Our experiment to run containers without mapping subuid/subgid, by using Seccomp User Notification: https://github.com/rootless-containers/subuidless
  - Idea to manage subuid/subgid using LDAP: https://github.com/shadow-maint/shadow/issues/154

- slirp is slow. lxc-user-nic could be used instead, but its `/etc/lxc/lxc-usernic` configuration is a mess.
  - Our experiment to bypass slirp using `SECCOMP_IOCTL_NOTIF_ADDFD` (kernel 5.9): https://github.com/rootless-containers/bypass4netns

- Can't mount block devices such as iSCSI
  - [LKL](https://lkl.github.io/) could be used to implement FUSE mounters for them

- NFS lacks support for user-namespaced CAP\_DAC\_OVERRIDE: https://www.redhat.com/sysadmin/rootless-podman-nfs
