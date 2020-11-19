---
title: User Namespaces
weight: 10
---

Rootless containers uses [`user_namespaces(7)`](http://man7.org/linux/man-pages/man7/user_namespaces.7.html)(UserNS) for
emulating fake privileges that are enough to create containers.

e.g. map UID 1000 to pseudo-root UID 0 in the UserNS:
```console
$ whoami
user1
$ id -u
1000
$ unshare --user --map-root-user
# cat /proc/self/uid_map 
         0       1000          1
# cat /proc/self/gid_map 
         0       1000          1
# id -u
0
```

The pseudo-root user gains capabilities such as `CAP_SYS_ADMIN` and `CAP_NET_ADMIN` inside UserNS
to perform fake-privileged operations such as creating mount namespaces, network namespaces, and creating TAP devices.

However, this user does not gain any actual privilege that can interfere with other users' files and processes.

## subuids & subgids, newuidmap & newgidmap

Just mapping the single pseudo-root UID/GID is not enough to run containers that require multiple UIDs and GIDs.
To map multiple UIDs and GIDs, Rootless Containers uses [SETUID](/glossary#setuid) binaries called `newuidmap` and `newgidmap`.
These binaries read configuration from files named `/etc/subuid` and `/etc/subgid`.

In the following example, 65,536 subuids (100000-165535) are allocated for a user named “user1”.
```console
$ cat /etc/subuid
user1:100000:65536
```

These subuids are mapped in the UserNS as follows:

Host UID | UserNS UID
---------|-----------
  1000   |     0
100000   |     1
100001   |     2
   ...   |   ...
165535   | 65536

The same applies to subgids defined in `/etc/subgid`.

Creating a user namespace with `newuidmap` and `newgidmap` binaries are slightly complicated.

[RootlessKit](/glossary#rootlesskit) provides a simple CLI that wraps these binaries:
```console
$ rootlesskit bash
# cat /proc/self/uid_map 
         0       1000          1
         1     100000      65536
# cat /proc/self/gid_map 
         0       1000          1
         1     100000      65536
```
