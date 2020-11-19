---
title: Incoming connections
weight: 30
---

Connections incoming from the Internet cannot directly reach network namespaces.

Two forwarder implementations are used for transferring connections to Rootless Containers' network namespaces.

## RootlessKit

The [RootlessKit](/glossary#rootlesskit) implementation is used by both [Docker/Moby](/getting-started/docker/) and [Podman](/getting-started/podman/).

## slirp4netns

[slirp4netns](/glossary#slirp4netns) also has its own port forwarder.

The slirp4netns implementation is slower than RootlessKit, 
however, the slirp4netns implementation can keep source IP addresses,
while the RootlessKit implementation treats all connections as if they were from 127.0.0.1.

The slirp4netns implementation is often preferred for recording access logs of public Web servers.

Docker/Moby and Podman optionally support using slirp4netns port forwarder.
