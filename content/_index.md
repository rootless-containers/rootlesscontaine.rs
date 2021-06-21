+++
title = "Rootless Containers"
draft = false

+++

Rootless containers refers to the ability for an unprivileged user to create,
run and otherwise manage containers. This term also includes the variety of
tooling around containers that can also be run as an unprivileged user. 

"Unprivileged user" in this context refers to a user who does not have any
administrative rights, and is "not in the good graces of the administrator" (in
other words, they do not have the ability to ask for more privileges to be
granted to them, or for software packages to be installed).

Pros:
* Can mitigate potential container-breakout vulnerabilities (Not a panacea, of course)
* Friendly to shared machines, especially in HPC environments

Cons:
* Complexity

See also [Caveats and Future work](/caveats).

## What are Rootless Containers and what are not?

When we say Rootless Containers, it means running the entire container runtime
as well as the containers without the root privileges.

Even when the containers are running as non-root users, when the runtime is still
running as root, we don't call them Rootless Containers.

While we allow using setuid (and/or setcap) binaries for some essential configurations
such as [`newuidmap`](./how-it-works/userns), when a larger part of the runtime is running with
setuid, we don't call it Rootless Containers.  We also don't call it Rootless Containers
when the root user inside a container is mapped to the root user outside the container.

### Examples of Rootless Containers

- [Docker rootless-mode](./getting-started/docker)
- [Podman rootless-mode](./getting-started/podman)
- [BuildKit rootless-mode](./getting-started/buildkit)
- [LXC unprivileged-mode](./getting-started/lxc)
- [Singularity userns-mode or fakeroot-mode](./getting-started/singularity)

Click the links for tutorials.

### *Non*-examples of Rootless Containers
- Allowing a non-root user to access to `/var/run/docker.sock`, by adding the user to `docker` group (`sudo usermod -aG docker somebody`)
- `docker run --user somebody`
- Docker userns-remap mode (`dockerd --userns-remap`)
- Podman userns-remap mode (`podman run --uidmap`)
- Kaniko
- Makisu
- LXD unprivileged-mode
- Singularity setuid-mode

<!--
TODO: do we consider UML to be an implementation of Rootless Containers or not?
At least it is very different from UserNS-based Rootless Containers...
-->
