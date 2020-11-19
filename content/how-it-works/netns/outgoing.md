---
title: Outgoing connections
weight: 20
---

Unsharing network namespaces isn't only for assigning IP addresses;
it is also essential to protect abstract UNIX sockets on the host from containerized processes.

However, unsharing network namespaces for Rootless Containers isn't straightforward,
because vEth pairs cannot be created across [UserNS](../../userns/) boundaries without the privilege.

LXC uses a [SETUID](/glossary#setuid) binary called `lxc-user-nic` for setting up vEth pairs.

Other implementations including Docker/Moby and Podman typically use TAP devices instead of vEth pairs,
and run a usermode network stack called [slirp](/glossary#slirp), which translates Ethernet packets 
to unprivileged socket system calls.

## SETUID binary

LXC uses a SETUID binary called `lxc-user-nic` for setting up vEth pairs.

Executing `lxc-user-nic` needs configuration per user to be added in [`/etc/lxc/lxc-usernet`](https://linuxcontainers.org/lxc/manpages/man5/lxc-usernet.5.html).

`lxc-user-nic` is also experimentally supported by [RootlessKit](/glossary#rootlesskit)
which is used by several projects including Docker/Moby and BuildKit.

## slirp

Several slirp implementations are used by Rootless Containers:

- [slirp4netns](/glossary#slirp4netns)
- [VPNKit](/glossary#vpnkit)

[slirp4netns is known to have significantly better throughput than VPNKit.](https://github.com/rootless-containers/slirp4netns#benchmarks)

Docker/Moby uses slirp4netns by default when slirp4netns is installed.
Otherwise falls back to VPNKit.

Podman only supports slirp4netns.
