---
title: Incoming connections
weight: 30
---

Connections incoming from the Internet cannot directly reach network namespaces.

Several forwarder implementations are used for transferring connections to Rootless Containers' network namespaces.

## RootlessKit

The [RootlessKit](/glossary#rootlesskit) implementation is used by both [Docker/Moby](/getting-started/docker/) and [Podman](/getting-started/podman/).

## slirp4netns

[slirp4netns](/glossary#slirp4netns) also has its own port forwarder.

The slirp4netns implementation is slower than RootlessKit, 
however, there was a benefit of using slirp4netns until the release of RootlessKit v3.0:
slirp4netns supported propagating the original source IP address of incoming connections,
while RootlessKit did not at that time.

The slirp4netns implementation was often preferred for recording access logs of public Web servers.

Docker/Moby and Podman (prior to v6.0) optionally support using slirp4netns port forwarder.

## pasta

[pasta](/glossary#pasta) also has its own port forwarder.

The pasta implementation is planned to be used by Podman as the default port forwarder in v6.0: https://github.com/containers/container-libs/pull/755

The pasta implementation is available for Docker/Moby as well.
