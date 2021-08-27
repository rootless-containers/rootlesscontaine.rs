---
title: Login
weight: 10
---

Most Rootless Containers implementations need the `$XDG_RUNTIME_DIR` environmental variable to be set.
When the environment variable is not set, features related to systemd and cgroups are unlikely to work properly.

The value is typically set to `/run/user/$UID` automatically by systemd or elogind on logging into the host.

Run the following command to confirm:
```console
$ echo $XDG_RUNTIME_DIR
/run/user/1000
```

The `$XDG_RUNTIME_DIR` environmental variable is set when:
* Logged in as a non-root user via the graphic console .
* Logged in as a non-root user via `ssh <user>@<hostname>` .
* Logged in as the root, and then switched to a non-root user via `machinectl shell <user>@` .

The environmental variable is _not_ set when:
* Logged in as the root, and then switched to a non-root user via `su -l <user>`
* Logged in as the root, and then switched to a non-root user via `sudo -u <user>`

{{< hint info >}}
**TL;DR**

Don't use `su` and `sudo` for switching from root to non-root.

Use `machinectl shell <user>@` or `ssh <user>@localhost` instead.
{{< /hint>}}

## [Optional] Start the systemd user session on boot

To run containers automatically on system start-up, the following command needs to be executed.

```console
$ sudo loginctl enable-linger $(whoami)
```

## [Optional] Enable dbus user session

Enabling dbus user session is typically needed for using systemd and cgroup v2.
Otherwise runc may fail with an error like `read unix @->/run/systemd/private: read: connection reset by peer: unknown.`

```console
$ systemctl --user is-active dbus
active
```

{{< tabs "sysctl" >}}
{{< tab "apt-get" >}}

```console
$ sudo apt-get install -y dbus-user-session
```
{{< /tab >}}
{{< tab "dnf" >}}

```console
$ sudo dnf install -y dbus-daemon
```
{{< /tab >}}
{{< /tabs >}}

In most cases, the dbus user session should be automatically enabled after installing the package above and relogging in.
If not, try running `systemctl --user enable --now dbus`.
