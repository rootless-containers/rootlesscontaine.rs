---
title: Singularity
weight: 50
---

{{< hint info >}}
**Note**

Please read [the common steps](../common) first.
{{< /hint>}}

Singularity provides two modes for running containers as a non-root user:
1. SETUID mode
2. Fakeroot mode

The former one does _not_ fall into the scope of Rootless Containers, as the
actual runtime binary gains the root privileges via the SETUID bit.

The latter one falls into our scope, but it should be noted that it does not
support creating [network namespaces](../../how-it-works/netns/) with Internet connectivity.

This means that you can't protect the abstract sockets on the host (such as D-Bus sockets)
from being connected from containerized processes, unless you disable Internet connectivity.

## Singularity fake-root mode
See https://sylabs.io/guides/3.6/user-guide/fakeroot.html
