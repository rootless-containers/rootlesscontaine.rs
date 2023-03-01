---
title: Apptainer/Singularity
weight: 50
aliases:
  - singularity
---

{{< hint info >}}
**Note**

Please read [the common steps](../common) first.
{{< /hint>}}

Apptainer (formerly known as Singularity) provides three modes for running containers as a non-root user:
1. User namespace mode (the default since version 1.1.0, or `apptainer exec --userns` with suid installation)
2. Fakeroot mode (`apptainer exec --fakeroot`)
3. Setuid mode (install extra apptainer-suid component)

The first mode does not use setuid root, so it is in the scope of Rootless Containers.
In fact, it does not even use a privileged helper the way other container systems' rootless modes do.
As a result, it has only one user id
directly mapped to the same user id outside the container, and all other user ids including root
are mapped to a nobody user id. 
This limits the types of applications it can run and does not meet OCI container requirements
because it cannot run many system-level applications.
Apptainer's focus is on application-level containers, not system-level containers.

The second mode also falls into our scope in that it can avoid its own setuid root runtime.
If `/etc/subuid` and `/etc/subgid` mappings will be set up, it will use only `newuidmap` and `newgidmap`. 
If they are not set up it will avoid even those privileged helpers and just use an unprivileged root-mapped namespace
combined with the unprivileged `fakeroot` command.
This fakeroot mode is primarily used for building containers to fool programs into thinking they have
multiple user ids available when they really don't.
It should be noted that Apptainer does not
support creating [network namespaces](../../how-it-works/netns/) with Internet connectivity.
This means that you can't protect abstract sockets on the host (such as D-Bus sockets)
from being connected from containerized processes.
This limitation also applies to the first mode but it doesn't matter for the types of applications that
Apptainer focuses on.
In fact the Apptainer project advises disabling network namespaces in order to reduce exposure
to kernel CVEs for exploits using unprivileged user namespaces combined with network namespaces.

The third mode does _not_ fall into the scope of Rootless Containers.  In that mode,
the optional runtime binary gains root privileges via the setuid bit and maps the root user
inside the container to the root user outside the container.  Although the setuid portion
of the runtime is kept fairly small, it mounts the contents of a container file as root
and has options to run overlayfs as root (for example `apptainer exec --writable`).
This mode is not enabled by default, but is available with a compilation and/or installation
option for those who don't want to enable unprivileged user namespaces or who want one
of the features that are only available in this mode (such as encrypted image files).

## Apptainer unprivileged user namespace mode
See https://apptainer.org/docs/admin/main/user_namespace.html#unprivileged-installations
## Apptainer fakeroot mode
See https://apptainer.org/docs/user/main/fakeroot.html

## SingularityCE fork

There is also an open source fork of the project (from before it was renamed to Apptainer) called
[SingularityCE](https://github.com/sylabs/singularity).
It has the same three modes as Apptainer, but setuid mode is the default.
Also the only fakeroot mode it supports is the one using `/etc/subuid` and `/etc/subgid` mappings,
not the mode using the `fakeroot` command.
