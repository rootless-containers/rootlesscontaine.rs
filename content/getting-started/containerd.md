---
title: containerd (nerdctl)
weight: 30
---

[nerdctl](https://github.com/containerd/nerdctl) is a Docker-compatible CLI for containerd.

nerdctl comes with helper scripts for running rootless containerd.

To run rootless containerd without nerdctl, see https://github.com/containerd/containerd/blob/master/docs/rootless.md

## Installation

{{< hint info >}}
**Note**

Please read [the common steps](../common) first.

Especially, make sure [`$XDG_RUNTIME_DIR`](../common/login/) to be set properly.
{{< /hint>}}


Download `nerdctl-full-<VERSION>-linux-amd64.tar.gz` from https://github.com/containerd/nerdctl/releases , and extract the
archive onto `/usr/local` (system-wide) or `~/.local` (home).

Then, run `containerd-rootless-setuptool.sh install` to set up the systemd unit:
```console
$ containerd-rootless-setuptool.sh install
[INFO] Checking RootlessKit functionality
[INFO] Checking cgroup v2
[INFO] Checking overlayfs
[INFO] Requirements are satisfied
[INFO] Creating "/home/exampleuser/.config/systemd/user/containerd.service"
[INFO] Starting systemd unit "containerd.service"
+ systemctl --user start containerd.service
+ sleep 3
+ systemctl --user --no-pager --full status containerd.service
● containerd.service - containerd (Rootless)
     Loaded: loaded (/home/exampleuser/.config/systemd/user/containerd.service; disabled; vendor preset: enabled)
    Drop-In: /home/exampleuser/.config/systemd/user/containerd.service.d
             └─override.conf
     Active: active (running) since Fri 2021-04-23 16:17:57 JST; 3s ago
   Main PID: 36976 (rootlesskit)
      Tasks: 30
     Memory: 21.1M
        CPU: 165ms
     CGroup: /user.slice/user-1001.slice/user@1001.service/app.slice/containerd.service
             ├─36976 rootlesskit --state-dir=/run/user/1001/containerd-rootless --net=slirp4netns --mtu=65520 --slirp4netns-sandbox=auto --slirp4netns-seccomp=auto --disable-host-loopback --port-driver=builtin --copy-up=/etc --copy-up=/run --copy-up=/var/lib --propagation=rslave /usr/local/bin/containerd-rootless.sh
             ├─36988 /proc/self/exe --state-dir=/run/user/1001/containerd-rootless --net=slirp4netns --mtu=65520 --slirp4netns-sandbox=auto --slirp4netns-seccomp=auto --disable-host-loopback --port-driver=builtin --copy-up=/etc --copy-up=/run --copy-up=/var/lib --propagation=rslave /usr/local/bin/containerd-rootless.sh
             ├─37007 slirp4netns --mtu 65520 -r 3 --disable-host-loopback --enable-sandbox --enable-seccomp 36988 tap0
             └─37014 containerd

Apr 23 16:17:57 examplemachine containerd-rootless.sh[37014]: time="2021-04-23T16:17:57.196395748+09:00" level=info msg="loading plugin \"io.containerd.grpc.v1.introspection\"..." type=io.containerd.grpc.v1
Apr 23 16:17:57 examplemachine containerd-rootless.sh[37014]: time="2021-04-23T16:17:57.196448426+09:00" level=info msg="Start subscribing containerd event"
Apr 23 16:17:57 examplemachine containerd-rootless.sh[37014]: time="2021-04-23T16:17:57.196655545+09:00" level=info msg="Start recovering state"
Apr 23 16:17:57 examplemachine containerd-rootless.sh[37014]: time="2021-04-23T16:17:57.196784184+09:00" level=info msg="Start event monitor"
Apr 23 16:17:57 examplemachine containerd-rootless.sh[37014]: time="2021-04-23T16:17:57.196798860+09:00" level=info msg="Start snapshots syncer"
Apr 23 16:17:57 examplemachine containerd-rootless.sh[37014]: time="2021-04-23T16:17:57.196807353+09:00" level=info msg="Start cni network conf syncer"
Apr 23 16:17:57 examplemachine containerd-rootless.sh[37014]: time="2021-04-23T16:17:57.196811520+09:00" level=info msg="Start streaming server"
Apr 23 16:17:57 examplemachine containerd-rootless.sh[37014]: time="2021-04-23T16:17:57.196819091+09:00" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
Apr 23 16:17:57 examplemachine containerd-rootless.sh[37014]: time="2021-04-23T16:17:57.196843099+09:00" level=info msg=serving... address=/run/containerd/containerd.sock
Apr 23 16:17:57 examplemachine containerd-rootless.sh[37014]: time="2021-04-23T16:17:57.196853937+09:00" level=info msg="containerd successfully booted in 0.037213s"
+ systemctl --user enable containerd.service
Created symlink /home/exampleuser/.config/systemd/user/default.target.wants/containerd.service → /home/exampleuser/.config/systemd/user/containerd.service.
[INFO] Installed "containerd.service" successfully.
[INFO] To control "containerd.service", run: `systemctl --user (start|stop|restart) containerd.service`
[INFO] To run "containerd.service" on system startup automatically, run: `sudo loginctl enable-linger exampleuser`
[INFO] ------------------------------------------------------------------------------------------
[INFO] Use `nerdctl` to connect to the rootless containerd.
[INFO] You do NOT need to specify $CONTAINERD_ADDRESS explicitly.
```

## Usage

```console
$ nerdctl run hello-world
```

Unlike rootless Docker, you do *NOT* need to specify `$CONTAINERD_ADDRESS` explicitly.

To start/stop the daemon, use `systemctl --user <start|stop> containerd` instead of `systemctl <start|stop> containerd`.
The systemd unit file is located as `~/.config/systemd/user/containerd.service`.

## Tips
### Enabling resource limitations (`nerdctl run --cpus`, `nerdctl run --memory`, ...)

Resource-related flags of `nerdctl run`, such as `--cpus`, `--memory`, `--blkio-weight`, and `--pids-limit` can be used only when the following conditions are satisfied:
* containerd version is 1.4 or later
* runc version is 1.0-rc91 or later
* The host is running with [cgroup v2](/getting-started/common/cgroup2)
* The host is running with systemd

To impose resource limitations without cgroup, see https://docs.docker.com/engine/security/rootless/#limiting-resources

### Changing the port forwarder

containerd uses [RootlessKit](/glossary#rootlesskit) as the default port forwarder.

However, as explained in [How it works](/how-it-works/netns/incoming/), sometimes
slirp4netns port forwarder is preferred over RootlessKit port forwarder.

To change the port forwarder to slirp4netns, create `~/.config/systemd/user/containerd.service.d/override.conf` with the following content:

```
[Service]
Environment="CONTAINERD_ROOTLESS_ROOTLESSKIT_PORT_DRIVER=slirp4netns"
```

And then restart the daemon:

```console
$ systemctl --user daemon-reload 
$ systemctl --user restart containerd
```

### Starting containers on boot

You need to run `sudo loginctl enable-linger ...`. See [Getting Started/Login](/getting-started/common/login/).

### Resetting to factory settings

Run the following commands to remove all containers and configurations:
```console
$ containerd-rootless-setuptool.sh uninstall
$ rootlesskit rm -rf ~/.local/share/containerd ~/.local/share/nerdctl ~/.config/containerd
```

## Further information
See https://github.com/containerd/nerdctl/blob/master/docs/rootless.md
