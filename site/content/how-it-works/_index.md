---
title: How it works
weight: 20
---

This section explains how Rootless Containers work under the hood.


- [User Namespaces](./userns): for emulating root privileges that are needed for running containers
- [Network Namespaces](./netns): for isolating network connections and IPC sockets
- [OverlayFS](./overlayfs): for deduplicating files
- [Cgroups](./cgroups): for limiting consumption of CPUs, memory, IO, and PIDs.
