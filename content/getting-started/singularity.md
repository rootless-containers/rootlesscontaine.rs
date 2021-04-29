---
title: Singularity
weight: 50
---

{{< hint info >}}
**Note**

Please read [the common steps](../common) first.
{{< /hint>}}

Singularity provides three modes for running containers as a non-root user:
1. Setuid mode
2. User namespace mode (`singularity exec --userns`)
3. Fakeroot mode (`singularity exec --fakeroot`)

The first one does _not_ fall into the scope of Rootless Containers.  In that mode,
the runtime binary gains root privileges via the setuid bit and maps the root user
inside the container to the root user outside the container.  Although the setuid portion
of the runtime is kept fairly small, it mounts the contents of a container file as root
and has options to run overlayfs as root (for example `singularity exec --writable`).

The second mode does not use setuid root, so it is in the scope of Rootless Containers.
In fact, it does not even use a setuid root helper.  As a result, it has only one user id
directly mapped to the same user id outside the container, and all other user ids including root
are mapped to a nobody user id. 
This limits the types of applications it can run and does not meet OCI container requirements
because it cannot run many system-level applications.

The third mode also falls into our scope in that it can avoid its own setuid root runtime
and use only `newuidmap` and `newgidmap`.  It should be noted however that it does not
support creating [network namespaces](../../how-it-works/netns/) with Internet connectivity.
This means that you can't protect abstract sockets on the host (such as D-Bus sockets)
from being connected from containerized processes, unless you disable Internet connectivity.
This limitation also applies to the second mode.

## Singularity unprivileged user namespace mode
See https://sylabs.io/guides/3.7/admin-guide/user_namespace.html#unprivileged-installations
## Singularity fake-root mode
See https://singularity.hpcng.org/user-docs/master/fakeroot.html
