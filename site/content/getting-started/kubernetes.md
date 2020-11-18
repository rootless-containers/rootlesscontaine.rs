---
title: Kubernetes
weight: 90
---

{{< hint info >}}
**Note**

Please read [the common steps](../common) first.
{{< /hint>}}

We have been proposing our patchset for supporting Rootless mode to the Kubernetes upstream,
but our proposal has not been merged yet: https://github.com/kubernetes/enhancements/pull/1371

## Usernetes

[Usernetes](https://github.com/rootless-containers/usernetes) is our reference Kubernetes distribution to support Rootless mode.

See https://github.com/rootless-containers/usernetes

```console
$ tar xjvf usernetes-x86_64.tbz
$ cd usernetes
$ ./install.sh --cri=containerd --cgroup-manager=systemd
$ export KUBECONFIG="$HOME/.config/usernetes/master/admin-localhost.kubeconfig"
$ kubectl apply -f manifests/*.yaml
```

## k3s

[k3s](https://k3s.io) supports Rootless mode using our Usernetes patchset.

```console
$ k3s server --rootless
```

See also https://rancher.com/docs/k3s/latest/en/advanced/#running-k3s-with-rootlesskit-experimental
