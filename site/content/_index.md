+++
date = "2017-04-07T21:07:40+10:00"
title = "Rootless Containers"
draft = false

+++

## Overview ##

Rootless containers refers to the ability for an unprivileged user to create,
run and otherwise manage containers. This term also includes the variety of
tooling around containers that can also be run as an unprivileged user. The
goal of this website it catalogue all of the open questions and unresolved
issues that need to be solved to bring us closer to rootless containers "for
all".

"Unprivileged user" in this context refers to a user who does not have any
administrative rights, and is "not in the good graces of the administrator" (in
other words, they do not have the ability to ask for more privileges to be
granted to them, or for software packages to be installed).

### Objectives ###

The objectives of this project are to allow an unprivileged user to:

* (**`O0`**) Create, run, and manage containers on their local machine.
* (**`O1`**) Create, modify, distribute, and extract container images on their
  local machine so they may run said container images though (**`O0`**).
* (**`O2`**) Create, manage, and use a container orchestrator on either their
  local machine (effective a single-node cluster) or on a set of machines they
  would ordinarily be able to communicate with and run unprivileged programs
  on.
<!-- If you're reading this and want to add to the above list, please open a PR! -->

While security is not a primary objective of this project (the goal is user
enablement), the intention is that any kernel-level improvements required to
facilitate this project will involve collaboration with the [Linux Container
Hardening Project][linux-ch]. We hope that the end result is that rootless
containers will be secure as well as useful, thus allowing enterprises to
support this usage of containers.

[linux-ch]: https://containerhardening.org/

## Status ##

This is a living document, as is maintained by humans. If you wish to add more
entries to this list, or have an update on a particular section, please [open a
pull request][gh].

[gh]: https://github.com/cyphar/rootlesscontaine.rs

### (`O0`) Runtime ###

* <input type="checkbox" disabled checked> Support rootless containers in a
  standardised container runtime.
* <input type="checkbox" disabled> Support checkpointing and restoring
  containers in a standard container runtime. This is separate from the
  "standardised container runtime" requirement because it is an auxilliary
  feature, and is a more generic problem.
* <input type="checkbox" disabled> Support rootless containers in an
  orchestration engine's container runtime. "Container runtime" in this context
  contains more features and tasks than the technically precise definition
  would include.
* <input type="checkbox" disabled> Write some documentation for users about how
  they can use rootless containers. It would be really nice if we had some
  allegory for Jessie's awesome [`dockerfiles`][jessie-dockerfiles] repo but
  using rootless containers (she [actually uses them as well][jessie-rootless]
  but some more descriptive documentation would really help).

The de-facto implementation of the Open Container Initiative's runtime
specification, [`runc`][runc], supports rootless containers [out of the
box][runc-rootless]. The main restrictions are the following:

* `cgroups` are not supported, because on the kernel side, unprivileged subtree
  management has not been implemented. After some discussions with the
  maintainer of `cgroups` is appears that there are some fundamental
  disagreements on the importance of unprivileged subtree management (and some
  of the API decisions made in the past make it difficult to implement). This
  restricts how many things a user can do with containers (`pause` and `resume`
  are not implemented, for example).

* [`criu`][criu] supports unprivileged dumping (`checkpoint`) but `restore` is
  not supported and the `checkpoint`ing aspects have not been well tested in
  the context of rootless containers.

* Network namespaces aren't used in most cases because currently unprivileged
  network namespaces are not very useful (in the standard usage of containers)
  because a new network namespace doesn't have any network interfaces (other
  than `lo`). The standard way of linking network namespaces (`veth` bridges)
  requires privileges in the host, which is not available in rootless
  containers. Some ideas for solving this are to implement unprivileged `veth`
  bridges in the kernel, or to implement some sort of unprivileged userspace
  `veth` bridge implementation. The jury is out on this one at the moment, so
  we just use the host's network namespace and call it a day.

[jessie-dockerfiles]: https://github.com/jessfraz/dockerfiles
[jessie-rootless]: https://blog.jessfraz.com/post/ultimate-linux-on-the-desktop/
[runc]: https://github.com/opencontainers/runc
[runc-rootless]: https://github.com/opencontainers/runc/pull/774
[criu]: https://criu.org/Main_Page

### (`O1`) Images ###

* <input type="checkbox" disabled checked> Add support to a tool for creating
  and modifying standardised container images as an unprivileged user.
* <input type="checkbox" disabled checked> Add support to a tool for extracting
  and manipulating a root filesystem extracted from a standardised container
  image as an unprivileged user.
* <input type="checkbox" disabled checked> Add support to a tool for
  distributing standardised container images as an unprivileged user. Note that
  currently the Open Container Initiative **does not** define a distribution
  format, so "distribution" refers to effectively doing `rsync` or something
  similar. Once distribution is standardised, this requirement will be
  extended.
* <input type="checkbox" disabled> Add support to a tool for building
  standardised container images as an unprivileged user.
  - This is currently [in progress](https://github.com/cyphar/orca-build/issues/6).
* <input type="checkbox" disabled> Add support to a storagedriver
  implementation to operate as an unprivileged user. This is separate from
  "extract and manipulate" because the naive implementation (just dump
  everything to a directory) works in principle but is not as efficient as
  using filesystem features to reduce the footprint of extracted images.

These requirements are implemented by several different tools, that each solve
a piece of the puzzle. These tools can then be combined to create a
"full-fledged" implementation of an image store or handler.

Distribution is the simplest problem to solve in this respect, and is
accomplished by [`skopeo`][skopeo]. In principle an Open Container Initiative
image can just be copied with `rsync`, but `skopeo` also has several features
related to conversion between different image formats that makes it incredibly
useful for a variety of other tasks. `skopeo` does not require privileges for
any of this core functionality (and in fact, is normally used as an
unprivileged user).

Extraction, modification, and creation are implemented by [`umoci`][umoci]. The
main issue with these operations is the limitations of the filesystem as an
unprivileged user, since there are a wide variety of operations that are not
permitted (for security reasons). As a result, `umoci`'s rootless support comes
with some caveats:

* While `umoci` will recreate the precise (or as close as possible) root
  filesystem state of the container image as a privileged user, as an
  unprivileged user certain hacks are required. Examples include creating dummy
  files in place of device nodes that an unprivileged user cannot create, and
  forcing everything to be owned by the current unprivileged user. As a result,
  the results of the image extraction may be counter-inuitive, but a lot of
  work has gone into reducing the weirdness of these hacks. In addition, `umoci
  repack` is implemented in such a way that it should not (in normal usage)
  result in an image that has `umoci`'s hacks baked into it (if you run an
  image modified by an unprivileged user as a privileged user it should work
  normally).

* Because of how discretionary access control works, `umoci` may have to modify
  the root filesystem of an extracted image when doing "read-only" operations
  on the extracted image. This means that root filesystems on `read-only` media
  (while `umoci` is operating on them) may run into issues.

[skopeo]: https://github.com/projectatomic/skopeo
[umoci]: https://github.com/openSUSE/umoci

### (`O2`) Orchestration ###

Orchestration is currently the biggest unknown. It's not quite clear what is
complete list of tasks necessary in order to make a container orchestrator run
with the majority of its functionality working as an unprivileged user.

* <input type="checkbox" disabled> Implement a way for a container networking
  interface plugin to run as an unprivileged user. This may involve just
  creating alternative plugins and swapping out the privileged versions.
<!-- TODO: Extend these. -->

These tasks do not have much significant progress. However, Cloud Foundry have
implemented [experimental rootless container support][cf-rootless]. The
downside to their implementation is that it still requires several privileged
binaries (thus not making it truly rootless, but it's a great first step).

[cf-rootless]: https://github.com/cloudfoundry/garden-runc-release/blob/b58ad52d3bc930d79bfe7318a910c772c675f0c6/docs/rootless-containers.md

## Prior Art ##
<!-- If we've missed your project, feel free to open a PR! -->

There has been some work in this space before, but (to our knowledge) nothing
this ambitious. The following are a list of projects that have some kind of
rootless container support.

* [`bubblewrap`][bubblewrap] (formerly `xdg-app`) implements parts of a
  container runtime as an unprivileged user, but to our knowledge it doesn't
  implement enough of the Open Container Initiative standard (even with
  [`bwrap-oci`][bwrap-oci]) to be a full container runtime.

* [`lxc`][lxc] has implemented quite a substantial portion of rootless containers, but
  their implementation requires privileged helper binaries to set up `cgroups`
  and network interfaces. This is a reasonable compromise for a container
  runtime (and `runc`'s solution was to just ignore those usecases) but it
  still means that practically it's not possible to use `lxc` in a way that
  would be required by this project.

[bubblewrap]: https://github.com/projectatomic/bubblewrap
[bwrap-oci]: https://github.com/projectatomic/bwrap-oci
[lxc]: https://linuxcontainers.org/lxc/

## FAQ ##
<!-- If there is a question that hasn't been answered here, please open a PR! -->

### Who are we? ###

At the moment, a party of one. But if this sounds cool to you, feel free to
join me! I'm on [Mastodon](https://mastodon.social/@cyphar), and can be reached
at ([cyphar@cyphar.com](mailto:cyphar@cyphar.com)).
