---
title: Docker/Moby
weight: 20
---

Rootless Docker/Moby was implemented in 2018 following rootless runc, containerd, and BuildKit.
Rootless Docker has been merged to the Docker/Moby upstream since Docker 19.03.

Docker 19.03 provides almost full features for Rootless mode, including support
for port fowarding (`docker run -p`) and multi-container networking (`docker network create`),
but it doesn't support limiting resources with cgroup.

Docker 20.10 added support for limiting resources using cgroup v2.

## Installation

{{< hint info >}}
**Note**

Please read [the common steps](../common) first.

Especially, make sure [`$XDG_RUNTIME_DIR`](../common/login/) to be set properly.
{{< /hint>}}


The official installation script can be executed by a non-root user without `sudo`.

{{< tabs >}}
{{< tab "Stable" >}}
```console
$ curl -fsSL https://get.docker.com/rootless | sh
```
{{< /tab >}}
{{< tab "Test" >}}
```console
$ curl -fsSL https://get.docker.com/rootless | CHANNEL=test sh
```
{{< /tab >}}
{{< tab "Nightly" >}}
```console
$ curl -fsSL https://get.docker.com/rootless | CHANNEL=nightly sh
```
{{< /tab >}}
{{< tab "RPMs/DEBs" >}}
Docker 20.10 provides `docker-ce-rootless-extras` RPMs and DEBs that can be installed by the root for all the users on the host.

```console
$ curl -fsSL https://get.docker.com | sudo sh
$ sudo apt-get install -y docker-ce-rootless-extras
```

After installing RPMs/DEBS, run the following command as a non-root user to create the systemd user-instance unit:

```console
$ dockerd-rootless-setuptool.sh install
```

{{< /tab >}}
{{< /tabs >}}


## Usage

```console
$ export PATH=$HOME/bin:$PATH
$ export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
$ docker run hello-world
```

To start/stop the daemon, use `systemctl --user <start|stop> docker` instead of `systemctl <start|stop> docker`.
The systemd unit file is located as `~/.config/systemd/user/docker.service`.

## Tips
### Enabling resource limitations (`docker run --cpus`, `docker run --memory`, ...)

Resource-related flags of `docker run`, such as `--cpus`, `--memory`, `--blkio-weight`, and `--pids-limit` can be used only when the following conditions are satisfied:
* Docker/Moby version is 20.10 or later
* containerd version is 1.4 or later
* runc version is 1.0-rc91 or later
* The host is running with [cgroup v2](/getting-started/common/cgroup2)
* The host is running with systemd

To impose resource limitations without cgroup, see https://docs.docker.com/engine/security/rootless/#limiting-resources

### Changing the network stack
Docker/Moby uses [slirp4netns](/glossary#slirp4netns) as the default network stack if slirp4netns v0.4.0 or later is installed.
Otherwise it falls back to [VPNKit](/glossary#vpnkit).

If slirp4netns is not installed on your host, download [the official slirp4netns binary](https://github.com/rootless-containers/slirp4netns)
to `~/bin` so that Docker/Moby can pick it up automatically. The functionalities are same as VPNKit, but slirp4netns is known to have better throughput.
However, slirp4netns is not included in the Docker package because they did not want to distribute slirp4netns's GPL2 binary along with Apache License 2.0 binaries.

Docker/Moby also supports [lxc-user-nic SETUID binary](/glossary#lxc-user-nic) experimentally: https://docs.docker.com/engine/security/rootless/#changing-the-network-stack

### Changing the port forwarder

Docker/Moby uses [RootlessKit](/glossary#rootlesskit) as the default port forwarder.

However, as explained in [How it works](/how-it-works/netns/incoming/), sometimes
slirp4netns port forwarder is preferred over RootlessKit port forwarder.

To change the port forwarder to slirp4netns, add the following line to the `[Service]` section of `~/.config/systemd/user/docker.service`:
```
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_PORT_DRIVER=slirp4netns"
```

And then restart the daemon:

```console
$ systemctl --user daemon-reload 
$ systemctl --user restart docker
```

### Starting containers on boot

You need to run `sudo loginctl enable-linger ...`. See [Getting Started/Login](/getting-started/common/login/).

### Resetting to factory settings

Run the following commands to remove all containers and configurations:
```console
$ systemctl --user stop docker
$ systemctl --user disable docker
$ rm -f ~/.config/systemd/user/docker.service
$ ~/bin/rootlesskit rm -rf ~/.local/share/docker ~/.config/docker
```

To uninstall binaries, remove the following files under `~/bin`:
```
containerd
containerd-shim
containerd-shim-runc-v2
ctr
docker
docker-init
docker-proxy
dockerd
dockerd-rootless-setuptool.sh
dockerd-rootless.sh
rootlesskit
rootlesskit-docker-proxy
runc
vpnkit
```

## Further information
See https://docs.docker.com/engine/security/rootless/
