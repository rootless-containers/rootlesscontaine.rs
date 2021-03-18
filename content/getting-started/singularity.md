---
title: Singularity
weight: 50
---

{{< hint info >}}
**Note**

Please read [the common steps](../common) first.
{{< /hint>}}

Singularity provides three modes for running containers as a non-root user:
1. SETUID mode
2. Unprivileged user namespace mode
3. Fakeroot mode

The first two do _not_ fall into the scope of Rootless Containers.  In the first case,
the runtime binary gains the root privileges via the SETUID bit.  In the second case,
the container has no root user inside the container; it only has the original user id
directly mapped to the same user id outside the container, and all other user ids are
mapped to a nobody user id.  Technically that is indeed rootless, but it is not 
commonly included in the Rootless Container scope.

The third mode falls into our scope, but it should be noted that it does not
support creating [network namespaces](../../how-it-works/netns/) with Internet connectivity.
This means that you can't protect the abstract sockets on the host (such as D-Bus sockets)
from being connected from containerized processes, unless you disable Internet connectivity.

## Singularity unprivileged user namespace mode
See https://sylabs.io/guides/3.7/admin-guide/user_namespace.html#unprivileged-installations
## Singularity fake-root mode
See https://singularity.hpcng.org/user-docs/master/fakeroot.html
