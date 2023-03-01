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

Pre-generating all possible values for /etc/subuid and /etc/subgid, based on uid and gid, rather than the user 
and group names, is also possible. This can simplify shared management of shared computing environments
using LDAP/AD, while there is no standardized way to store or retrieve subuid and subgid values
from those directories.

An example python program to generate the files:

```python
with open("/etc/subuid", "w") as f:
    for uid in range(1000, 65536):
        f.write("%d:%d:65536\n" %(uid,uid*65536))

with open("/etc/subgid", "w") as f:
    for uid in range(1000, 65536):
        f.write("%d:%d:65536\n" %(uid,uid*65536))
```

When doing this, however, it's important to note that duplicate entries will be added to the files
when adding new local users or groups. Those new entries will be based on user name or group name.

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


## Advanced information
### Specific subuid range for systemd-homed users

When the user's home directory is managed by [`systemd-homed`](https://wiki.archlinux.org/title/Systemd-homed),
the subuid range has to be typically chosen from 524288-1878982656 (i.e., 0x80000-0x6fff0000).

{{< hint info >}}
**Am I using `systemd-homed` ?**

If you have `~/.identity` in your home directory, your home directory is probably managed by `systemd-homed`.

Otherwise your home directory is not managed by `systemd-homed` (even if `systemd-homed` process is running),
and you can just skip reading this section.

In 2023, no well-known Linux distribution seems using `systemd-homed` by default.
{{< /hint>}}

The following example allocates 65,536 subuids for 524288-589823 (0x80000-0x8ffff).
```
user1:524288:65536
```

To obtain the correct subuid range for `systemd-homed` users, run `userdbctl` and see the "begin container users" line
and the "end container users" line:

```console
$ userdbctl
   NAME                           DISPOSITION        UID   GID REALNAME                     HOME             SHELL
   root                           intrinsic            0     0 -                            /root            /bin/bash
...
┌─ ↓ begin container users ↓      container       524288     - First container user         -                -
└─ ↑ end container users ↑        container   1878982656     - Last container user          -                -
```

The range is decided on the compilation time of [systemd](https://github.com/systemd/systemd/blob/v253/meson_options.txt#L246-L249).
