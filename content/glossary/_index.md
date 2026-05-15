---
title: Glossary
weight: 80
---

[Open an issue](https://github.com/rootless-containers/rootlesscontaine.rs/issues) to request explaining more.

## C
#### Cgroup
Control groups, used for limiting resource usage of containers.

Using cgroups is considered to be optional for Rootless Containers.

## D
#### Delegation
In the context of [Cgroup](#cgroup), delegation means allowing non-root users to modify Cgroup configuration to impose
limits on consumption of CPU, memory, I/O, and PIDs.

Delegation typically needs cgroup v2 and systemd.

## F
#### FUSE
Filesystem in User Space.

While mounting kernel-mode filesystems inside [User Namespaces](#user-namespaces) is almost unsupported,
mounting FUSE is supported since kernel 4.18.

#### FUSE-OverlayFS
A [FUSE](#fuse) implementation of OverlayFS.

Web site: https://github.com/containers/fuse-overlayfs

FUSE-OverlayFS is mostly used for allowing rootless OverlayFS on old kernels (< 5.11).

See [How it works/OverlayFS](../how-it-works/overlayfs/).

## G
#### gvisor-tap-vsock
A [slirp](#slirp) implementation written in pure Go, based on [gVisor](https://gvisor.dev/)'s network stack.

Web site: https://github.com/containers/gvisor-tap-vsock

## N
#### Network Namespaces
See [How it works/Network Namespaces](../how-it-works/netns/).

#### newuidmap & newgidmap
See [How it works/User Namespaces](../how-it-works/userns/).

## L
### lxc-user-nic
See [How it works/Network Namespaces/Outgoing Connections](../how-it-works/netns/outgoing).

## P
#### pasta
Yet another [slirp](#slirp) implementation, focusing on performance.

pasta is used by Podman as the default networking mode since v5.0.

Web site: https://passt.top

## R
#### rootless-cni-infra
A sandbox container used for implementing multi-container networking for Podman (prior to v3.2).

Web site: https://github.com/containers/podman/tree/v3.1/contrib/rootless-cni-infra

#### Rootless Containers
When we say Rootless Containers, it means running the entire container runtime as well as the containers without the root privileges.

Even when the containers are running as non-root users, when the runtime is still running as the root, we don’t call them Rootless Containers.

See [the top page](/) for the further information.

#### RootlessKit
RootlessKit is a tool that helps setting up [User Namespaces](#user-namespaces) with [slirp](#slirp) and port forwarding.

Web site: https://github.com/rootless-containers/rootlesskit

## S
#### SETUID

A binary file with SETUID bit is executed as the file owner.
SETUID is typically used for allowing non-root users to execute specific binary with the root privileges.

Executing SETUID binaries is dangerous; Rootless Containers typically do not use any SETUID binary,
with a small number of exceptions such as `newuidmap` and `newgidmap`.

#### slirp
A technique to translate Ethernet packets to unprivileged TCP/IP socket syscalls.
Widely used by virtual machine implementations such as QEMU and Rootless Container implementations.

Several implementations are known:
- [slirp4netns](/glossary#slirp4netns)
- [VPNKit](/glossary#vpnkit)
- [pasta](/glossary#pasta)
- [gvisor-tap-vsock](/glossary#gvisor-tap-vsock)

Note that the modern usage of "slirp" has signifantly changed since the original release of "slirp" in 1995.
The original slirp in 1995 was used for connecting PCs w/o Ethernet devices to the Internet via a dial-up shell account,
by emulating SLIP/PPP.

As of 2020, "slirp" is rarely used in conjunction with SLIP/PPP.

See also [How it works/Network Namespaces](../how-it-works/netns/).

#### slirp4netns
Our implementation of [slirp](#slirp) optimized for Rootless Containers.
Used by Docker/Moby (when installed) and Podman (prior to v6.0).

Web site: https://github.com/rootless-containers/slirp4netns

See also [How it works/Network Namespaces](../how-it-works/netns/).

#### subuid & subgid
See [How it works/User Namespaces](../how-it-works/userns/).

## U
#### Unprivileged
In the context of containers, "unprivileged" can have (at least) three different meanings:
1. Running a Docker/Podman container without `docker run --privileged`, or running a Kubernetes container without `securityContext.privileged=true`
2. Running a LXC container as a non-root user (and by a non-root user, typically)
3. Running a LXD container as a non-root user but keep the LXD daemon running as the root

Only the second meaning falls into the scope of [Rootless Containers](#rootless-containers).

#### User Namespaces
See [How it works/User Namespaces](../how-it-works/userns/).

#### VPNKit
An OCaml implementation of [slirp](#slirp), used by Rootless Docker/Moby when [slirp4netns](#slirp4netns) is not installed.

Aside from Rootless Docker/Moby, VPNKit was historically used by Docker for Mac/Win for connecting the LinuxKit VM to the Internet.

See also [How it works/User Namespaces](../how-it-works/userns/).

Web site: https://github.com/moby/vpnkit

## X
#### XDG\_RUNTIME\_DIR
The equivalent of `/var/run` for non-root users.
Typically set to `/run/user/$UID`.

See [Getting Started/Common/Login](/getting-started/common/login/)
