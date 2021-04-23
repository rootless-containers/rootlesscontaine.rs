---
title: "[Optional] cgroup v2"
weight: 40
---

{{< hint info >}}
**Note**

Enabling cgroup v2 is optional.
{{< /hint>}}


Enabling cgroup v2 is often needed for running Rootless Containers with limiting the consumption of the CPU, memory, I/O, and PIDs resources,
e.g. `docker run --memory 32m`.

Note that cgroup is not needed for just limiting resources with traditional ulimit and [cpulimit](https://github.com/opsengine/cpulimit),
though they work in process-granularity rather than in container-granularity.
See [here](https://docs.docker.com/engine/security/rootless/#limiting-resources) for the further information.

## Checking whether cgroup v2 is already enabled

If `/sys/fs/cgroup/cgroup.controllers` is present on your system, you are using v2, otherwise you are using v1.

The following distributions are known to use cgroup v2 by default:
- Fedora (since 31)
- Arch Linux (since April 2021)

Debian GNU/Linux 11 (ETA: 2021) and [Ubuntu 21.10](https://bugs.launchpad.net/ubuntu/+source/snapd/+bug/1850667) are also planned to use cgroup v2 by default.

## Enabling cgroup v2

Enabling cgroup v2 for containers requires kernel 4.15 or later. Kernel 5.2 or later is recommended.

And yet, delegating cgroup v2 controllers to non-root users requires a recent version of systemd. systemd 244 or later is recommended.

To boot the host with cgroup v2, add the following string to the `GRUB_CMDLINE_LINUX` line in `/etc/default/grub` and then run `sudo update-grub`.
```
systemd.unified_cgroup_hierarchy=1
```

## Enabling CPU, CPUSET, and I/O delegation

By default, a non-root user can only get `memory` controller and `pids` controller to be delegated.
```console
$ cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/cgroup.controllers
memory pids
```

To allow delegation of other controllers such as `cpu`, `cpuset`, and `io`, run the following commands:

```console
$ sudo mkdir -p /etc/systemd/system/user@.service.d
$ cat <<EOF | sudo tee /etc/systemd/system/user@.service.d/delegate.conf
[Service]
Delegate=cpu cpuset io memory pids
EOF
$ sudo systemctl daemon-reload
```

Delegating `cpuset` is recommended as well as `cpu`. Delegating `cpuset` requires systemd 244 or later.
