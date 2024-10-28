---
title: "[Optional] sysctl"
weight: 50
---

{{< hint info >}}
**Note**

Configuring sysctl is optional for most distributions.
{{< /hint>}}

Old versions of Debian, Arch, and RHEL/CentOS are known to require reconfiguration of sysctl
to enable [User Namespaces](/how-it-works/userns/).

{{< tabs "sysctl" >}}
{{< tab "Debian GNU/Linux 10" >}}
{{< hint info >}}
**Note**

These steps are *not* needed for Debian 11.
These steps are also *not* needed for Ubuntu.
{{< /hint>}}
Create `/etc/sysctl.d/99-rootless.conf` with the following content:
```
kernel.unprivileged_userns_clone=1
```

Then run the following command to reload the new sysctl configuration:
```console
$ sudo sysctl --system
```
{{< /tab >}}
{{< tab "Arch Linux (old)" >}}
{{< hint info >}}
**Note**

These steps are *no longer* needed for Arch Linux as of April 2021.
{{< /hint>}}
Create `/etc/sysctl.d/99-rootless.conf` with the following content:
```
kernel.unprivileged_userns_clone=1
```
Then run the following command to reload the new sysctl configuration:
```bash
sudo sysctl --system
```
{{< /tab >}}
{{< tab "RHEL/CentOS 7">}}
{{< hint info >}}
**Note**

These steps are *not* needed for RHEL/CentOS 8.
{{< /hint>}}
Create `/etc/sysctl.d/99-rootless.conf` with the following content:
```
user.max_user_namespaces=28633
```
<!-- nobody knows the origin of the 28633 magic value, lol -->

Then run the following command to reload the new sysctl configuration:
```bash
sudo sysctl --system
```

{{< /tab >}}
{{< /tabs >}}


## Allowing ping
Most distributions do not allow non-root users to send ICMP Echo Request packets (aka `ping`) by default.

To allow running `ping` without root, create `/etc/sysctl.d/99-rootless.conf` with the following content:
```
net.ipv4.ping_group_range = 0 2147483647
```

Then run the following command to reload the new sysctl configuration:
```bash
sudo sysctl --system
```

## Allowing listening on TCP & UDP ports below 1024
Most distributions do not allow non-root users to listen on TCP & UDP ports below 1024.
e.g. listening on 80/tcp would fail with "permission denied", while listening on 8080/tcp would success.

To allow listening on any port without root, create `/etc/sysctl.d/99-rootless.conf` with the following content:
```
net.ipv4.ip_unprivileged_port_start=0
```

Then run the following command to reload the new sysctl configuration:
```bash
sudo sysctl --system
```

