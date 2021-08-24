---
title: Podman
weight: 21
---

The following table shows the feature implementation status of Rootless Podman:

| Version | Notable changes
|---------|---------------------------
| Pre-1.1 | Initial support for Rootless mode
|  1.1   | Added support for port forwarding (`podman run -p`)
|  1.5   | Added support for cgroup v2
|  2.1   | Added support for multi-container networking (`podman create network`)

{{< hint info >}}
**FAQ: Docker/Moby vs Podman?**

Until recently, Docker/Moby had lacked support for cgroup v2, and on the other hand
Podman had lacked support for multi-container networking.

As of October 2020, the two projects implement almost the same features with regard
to the support for Rootless mode.
{{< /hint>}}

## Installation
{{< hint info >}}
**Note**

Please read [the common steps](../common) first.
{{< /hint>}}

The easiest way to install Rootless Podman is to install `podman` package.
Requires `sudo`.

{{< tabs >}}
{{< tab "apt-get" >}}
```console
$ sudo apt-get install -y podman
```
{{< /tab >}}
{{< tab "dnf" >}}

```console
$ sudo dnf install -y podman
```
{{< /tab >}}
{{< /tabs >}}

Rootless Podman could also be installed without `sudo` in theory.
However, as of October 2020, there is no official Podman binaries that can be installed
without `sudo`.


## Usage

Just run `podman` command.

```console
$ podman run docker.io/library/hello-world
```

## Tips
### Enabling resource limitations (`podman run --cpus`, `podman run --memory`, ...)

Resource-related flags of `podman run`, such as `--cpus`, `--memory`, `--blkio-weight`, and `--pids-limit` can be used only when the following conditions are satisfied:
* Podman version is 1.5 later
* runc version is 1.0-rc91 or later, or crun is installed
* The host is running with [cgroup v2](/getting-started/common/cgroup2)
* The host is running with systemd

To impose resource limitations without cgroup, see https://docs.docker.com/engine/security/rootless/#limiting-resources (read `docker` as `podman`)

### Changing the port forwarder

Podman uses [RootlessKit](/glossary#rootlesskit) as the default port forwarder.

However, as explained in [How it works](/how-it-works/netns/incoming/), sometimes
slirp4netns port forwarder is preferred over RootlessKit port forwarder.

To change the port forwarder to slirp4netns, run `podman run` with `--network slirp4netns:port_handler=slirp4netns`.

See also http://docs.podman.io/en/latest/markdown/podman-run.1.html

### Starting containers on boot

As Podman lacks the central daemon, you need to create systemd unit files to launch the each of the
containers on the system startup.

See http://docs.podman.io/en/latest/markdown/podman-generate-systemd.1.html

Also, you need to run `sudo loginctl enable-linger ...`. See [Getting Started/Login](/getting-started/common/login/).

### Resetting to factory settings

Run the following commands to remove all containers and configurations:
```console
$ podman rm -f $(podman ps -a -q)
$ podman unshare rm -rf ~/.local/share/containers ~/.config/containers
```

To uninstall binaries, remove `podman` package with the package manager.

## Further information
See https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md
