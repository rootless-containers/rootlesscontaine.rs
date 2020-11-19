---
title: /etc/subuid and /etc/subgid
weight: 30
---

Rootless Containers implementations mostly expect `/etc/subuid` to contain at least 65,536 subuids.

In the following example, 65,536 subuids (100000-165535) are allocated for a user named "user1".

```console
$ cat /etc/subuid
user1:100000:65536
```

The same applies to subgids defined in `/etc/subgid`. See also [How it works/User Namespaces](/how-it-works/userns/).

These subuids and subgids are typically automatically configured by the system.

If subuids and subgids are not configured, you need to edit `/etc/subuid` and `/etc/subgid` directly with a text editor:

```console
$ sudo vi /etc/subuid
```

## newuidmap and newgidmap

`newuidmap` and `newgidmap` needs to be installed on the host.
These binaries are typically installed by default.

{{< tabs "uidmap" >}}
{{< tab "apt-get" >}}

```console
$ sudo apt-get install -y uidmap
```
{{< /tab >}}
{{< tab "dnf" >}}

```console
$ sudo dnf install -y shadow-utils
```
{{< /tab >}}
{{< /tabs >}}
