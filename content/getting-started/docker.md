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

Docker 23 further simplified the installation process.

Docker 29.5 added support for `--net=host` and improved the support for propagating the original source IP address of incoming connections.

## Installation

{{< hint info >}}
**Note**

Please read [the common steps](../common) first.

Especially, make sure [`$XDG_RUNTIME_DIR`](../common/login/) to be set properly.
{{< /hint>}}



{{< tabs >}}
{{< tab "With RPMs/DEBs" >}}
Docker (since 20.10) provides `docker-ce-rootless-extras` RPMs and DEBs that can be installed by the root for all the users on the host.
The package is usually automatically installed on installing Docker from <https://get.docker.com>.

```bash
curl -o install.sh -fsSL https://get.docker.com
sudo sh install.sh
```

After installing Docker, run the following command as a non-root user to create the systemd user-instance unit:

```bash
dockerd-rootless-setuptool.sh install
```

{{< /tab >}}
{{< tab "Without RPMs/DEBs" >}}
This method does not use RPMs/DEBS and can be executed by a non-root user without `sudo`.

```bash
curl -o rootless-install.sh -fsSL https://get.docker.com/rootless
sh rootless-install.sh
export PATH=$HOME/bin:$PATH
```
{{< /tab >}}
{{< /tabs >}}


## Usage

Starting with Docker v23, the `docker` CLI connects to the rootless daemon by default when it is available.

Run `docker info` to confirm that the `docker` CLI is connecting to the rootless daemon:

```console
$ docker info
Client: Docker Engine - Community
 Version:    28.3.3
 Context:    rootless
...
Server:
...
 Security Options:
  seccomp
   Profile: builtin
  rootless
  cgroupns
...
```

<details>
<summary>Historical information for Docker prior to v23</summary>
<p>

To connect to the rootless daemon, you had to set either the CLI context or an environment variable.

{{< tabs "docker-cli-config" >}}
{{< tab "CLI context (Modern)" >}}
```bash
docker context use rootless
docker run hello-world
```
{{< /tab >}}
{{< tab "Env var (Classic)" >}}
```bash
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
docker run hello-world
```
{{< /tab >}}
{{< /tabs >}}

</p>
</details>


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
Otherwise it falls back to [VPNKit](/glossary#vpnkit), [pasta](/glossary#pasta), or [gvisor-tap-vsock](/glossary#gvisor-tap-vsock) in that order.

To explicitly specify the network stack, create `~/.config/systemd/user/docker.service.d/override.conf` with the following content:

```
[Service]
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_NET=gvisor-tap-vsock"
```

And then restart the daemon:

```bash
systemctl --user daemon-reload
systemctl --user restart docker
```

Docker/Moby also supports [lxc-user-nic SETUID binary](/glossary#lxc-user-nic) experimentally: https://docs.docker.com/engine/security/rootless/#changing-the-network-stack

### Preserving source IP addresses of incoming connections

Docker/Moby uses [RootlessKit](/glossary#rootlesskit) as the default port forwarder,
but RootlessKit did not support propagating the original source IP address of incoming connections until RootlessKit v3.0 that is bundled with Docker v29.5.

For prior versions, you had to change the port forwarder to slirp4netns or pasta to keep source IP addresses of incoming connections.

{{< tabs "preserve-src-ip" >}}
{{< tab "v29.5 or later" >}}

RootlessKit v3.0 bundled with Docker v29.5 added the support for propagating the original source IP address of incoming connections, so there is no reason to use slirp4netns or pasta
port forwarder anymore.

However, the source IP propagation requires disabling the userland proxy:

```bash
mkdir -p ~/.config/docker
echo '{"userland-proxy": false}' >~/.config/docker/daemon.json
systemctl --user restart docker
```

You may also need to load the `br_netfilter` kernel module:

```bash
sudo tee /etc/modules-load.d/docker.conf <<EOF >/dev/null
br_netfilter
EOF
sudo systemctl restart systemd-modules-load.service
```

{{< /tab >}}
{{< tab "Prior versions (slirp4netns)" >}}
Create `~/.config/systemd/user/docker.service.d/override.conf` with the following content:

```
[Service]
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_PORT_DRIVER=slirp4netns"
```

And then restart the daemon:

```bash
systemctl --user daemon-reload
systemctl --user restart docker
```
{{< /tab >}}
{{< tab "Prior versions (pasta)" >}}
Create `~/.config/systemd/user/docker.service.d/override.conf` with the following content:

```
[Service]
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_NET=pasta"
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_PORT_DRIVER=implicit"
```

(The port driver is called "implicit" in RootlessKit, because the ports are handled by pasta, not RootlessKit itself.)

And then restart the daemon:

```bash
systemctl --user daemon-reload
systemctl --user restart docker
```
{{< /tab >}}
{{< /tabs >}}

### Starting containers on boot

You need to run `sudo loginctl enable-linger ...`. See [Getting Started/Login](/getting-started/common/login/).

### Resetting to factory settings

Run the following commands to remove all containers and configurations:
```bash
dockerd-rootless-setuptool.sh uninstall
~/bin/rootlesskit rm -rf ~/.local/share/docker ~/.config/docker
```

To uninstall binaries, run `sudo apt-get remove docker-ce-*`
or `sudo dnf remove docker-ce-*`.

If you installed Rootless Docker without using packages,
remove the following files under `~/bin`:
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
