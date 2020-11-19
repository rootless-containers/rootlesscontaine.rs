---
title: udocker
weight: 55
---

{{< hint info >}}
**Note**

Please read [the common steps](../common) first.
{{< /hint>}}

{{< hint warning >}}
**Note**

udocker is _not_ [Docker](../docker).
{{< /hint>}}

[udocker](https://github.com/indigo-dc/udocker/) provides four modes for running containers as a non-root user:
1. Pn: Proot mode
2. Fn: fakechroot mode
3. Rn: rootless [runc](../runc) mode
4. Sn: [Singularity](../singularity) mode

The third mode (Rn) falls into the scope of Rootless Containers, but the others are not:
Pn and Fn are not containers (in our view), and Sn requires SETUID bit.

For futher information, see https://github.com/indigo-dc/udocker/
