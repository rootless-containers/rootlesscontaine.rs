---
title: Kubernetes
weight: 90
---

{{< hint info >}}
**Note**

Please read [the common steps](../common) first.
{{< /hint>}}

Running node components of Kubernetes in a user namespace has been supported since Kubernetes v1.22 (alpha).

See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-in-userns

## kind

See https://kind.sigs.k8s.io/docs/user/rootless/

## Usernetes

[Usernetes](https://github.com/rootless-containers/usernetes) is our reference Kubernetes distribution to support Rootless mode.

See https://github.com/rootless-containers/usernetes

```console
$ tar xjvf usernetes-x86_64.tbz
$ cd usernetes
$ ./install.sh --cri=containerd
$ export KUBECONFIG="$HOME/.config/usernetes/master/admin-localhost.kubeconfig"
$ kubectl apply -f manifests/*.yaml
```

## k3s

[k3s](https://k3s.io) supports Rootless mode experimentally.

See https://rancher.com/docs/k3s/latest/en/advanced/#running-k3s-with-rootless-mode-experimental

## Manual deployment ("Hard way")

See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-in-userns/#userns-the-hard-way
