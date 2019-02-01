+++
date = "2017-04-07T21:07:40+10:00"
title = "Rootless Containers"
draft = false

+++

## Overview ##

Rootless containers refers to the ability for an unprivileged user to create,
run and otherwise manage containers. This term also includes the variety of
tooling around containers that can also be run as an unprivileged user. The
goal of this website is to catalogue all of the open questions and unresolved
issues that need to be solved to bring us closer to rootless containers "for
all".

"Unprivileged user" in this context refers to a user who does not have any
administrative rights, and is "not in the good graces of the administrator" (in
other words, they do not have the ability to ask for more privileges to be
granted to them, or for software packages to be installed).

<iframe src="https://www.slideshare.net/slideshow/embed_code/key/nJyXOTcSJaz0cL" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <a href="https://www.slideshare.net/AkihiroSuda/rootless-containers" title="Rootless Containers" target="_blank">Our talk at DevConf.cz 2019</a></div>

- - -
# :warning: OUTDATED CONTENTS :warning: #

The following contents are outdated and planned to be updated.

For a while, please refer to the slide deck above.

### Objectives ###

The objectives of this project are to allow an unprivileged user to:

* (**`O0`**) Create, run, and manage containers on their local machine.
* (**`O1`**) Create, modify, distribute, and extract container images on their
  local machine so they may run said container images through (**`O0`**).
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
* <input type="checkbox" disabled checked> Support emulating system calls
  that are not allowed in regular rootless containers.
  (e.g. `setgroups(2)`, `seteuid(2)`, `chown(2)`...)
  i.e. get `apt` and `yum` to work.
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

* `cgroups` are not supported in the general case, because on the kernel side,
  unprivileged subtree management has not been implemented. There is a new
  `nsdelegate` mount option for the **host** cgroup mount (which allows for
  sub-tree delegation of cgroupv2 when you create a cgroup namespace), but
  since `runc` supports neither cgroupv2 nor cgroup namespaces this feature is
  not useful to us at the moment. However, [recently][runc-pr1540] `runc` can
  opportunistically use cgroups if the system was configured to allow users to
  use them (see [`lxcfs`'s PAM module][lxcfs] for an example of how such a
  system can be configured).

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
  `veth` bridge implementation with [TAP][tap-in-unprivileged-netns]. The jury
  is out on this one at the moment, so we just use the host's network namespace
  and call it a day.

* Some system calls such as `setgroups(2)`, `seteuid(2)`, and `chown(2)` are
  [known not to work][aleksa-20160627] in rootless containers, unless multiple
  UID/GID mappings are configured with SUID utility binaries (`newuidmap(1)`,
  `newgidmap(1)`). Programs such as `apt` and `yum` are known to require such
  system calls to work.

 While `setgroups(2)` and `seteuid(2)` are only "temporary" for the process
  which executes them, syscalls like `chown(2)` and `mknod(2)` actually
  modify persistent storage and thus rootless container runtimes may wish
  to persist this data in a interoperable fashion. For this purpose we define
  [the `user.rootlesscontainers` `xattr` attribute][rootlesscontainers-proto].
  This is also useful for emulating "real root" with tools like [PRoot][proot]
  or [remainroot][remainroot]. [runROOTLESS][runrootless] provides a forked
  version of PRoot that implements the `user.rootlesscontainers` `xattr`
  attribute.

[jessie-dockerfiles]: https://github.com/jessfraz/dockerfiles
[jessie-rootless]: https://blog.jessfraz.com/post/ultimate-linux-on-the-desktop/
[runc]: https://github.com/opencontainers/runc
[runc-rootless]: https://github.com/opencontainers/runc/pull/774
[criu]: https://criu.org/Main_Page
[runc-pr1540]: https://github.com/opencontainers/runc/pull/1540
[lxcfs]: https://github.com/lxc/lxcfs
[aleksa-20160627]: https://www.cyphar.com/blog/post/20160627-rootless-containers-with-runc
[rootlesscontainers-proto]: https://github.com/rootless-containers/rootlesscontaine.rs/blob/f7b0f5486bb0e0be75771ce50c54471296f040eb/docs/proto/rootlesscontainers.proto
[proot]: https://proot-me.github.io
[remainroot]: https://github.com/cyphar/remainroot
[runrootless]: https://github.com/rootless-containers/runrootless
[tap-in-unprivileged-netns]: https://github.com/AkihiroSuda/runrootless/tree/f1c2e886d07b280ae1558d04cfe074aa6889a9a4/misc/vde

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
* <input type="checkbox" disabled checked> Add support to a tool for building
  standardised container images as an unprivileged user.
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
  the results of the image extraction may be counter-intuitive, but a lot of
  work has gone into reducing the weirdness of these hacks. In addition, `umoci
  repack` is implemented in such a way that it should not (in normal usage)
  result in an image that has `umoci`'s hacks baked into it (if you run an
  image modified by an unprivileged user as a privileged user it should work
  normally). However, `umoci` supports the usage of the previously mentioned
  [`user.rootlesscontainers` `xattr` attribute][rootlesscontainers-proto],
  which means that programs that accept `user.rootlesscontainers` attributes
  will be able to interoperate correctly.

* Because of how discretionary access control works, `umoci` may have to modify
  the root filesystem of an extracted image when doing "read-only" operations
  on the extracted image. This means that root filesystems on `read-only` media
  (while `umoci` is operating on them) may run into issues.

Building of images is implemented by [`orca-build`][orca-build], which is a
proof-of-concept wrapper around `runc`, `umoci`, and `skopeo`. There are
several other container builder projects which may implement this feature in
the future, but right now `orca-build` is fast enough and is quite minimal. It
also has the additional feature of being compatible with the `Dockerfile`
format for specifying build steps.

[skopeo]: https://github.com/projectatomic/skopeo
[umoci]: https://umo.ci/
[orca-build]: https://github.com/cyphar/orca-build

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
downside to their implementation is that it still requires privileged setup
steps, and a privileged networking binary during the container lifecycle
(thus not making it truly rootless, but it's a great first step).

[cf-rootless]: https://github.com/cloudfoundry/garden-runc-release/blob/43a4aaf771a18173f1b57a8aec749d16433ad2a7/docs/rootless-containers.md

## Prior Art ##
<!-- If we've missed your project, feel free to open a PR! -->

There has been some work in this space before, but (to our knowledge) nothing
this ambitious. The following are a list of projects that have some kind of
rootless container support.

* [`bubblewrap`][bubblewrap] (formerly `xdg-app`) implements parts of a
  container runtime as an unprivileged user, but to our knowledge it doesn't
  implement enough of the Open Container Initiative standard (even with
  [`bwrap-oci`][bwrap-oci]) to be a full container runtime.

* [`lxc`][lxc] has implemented quite a substantial portion of rootless
  containers, but their implementation requires privileged helper binaries to
  set up `cgroups` and network interfaces. This is a reasonable compromise for
  a container runtime (and `runc`'s solution was to just ignore those
  usecases), and `lxc` does have knobs to disable the use of those binaries,
  but it still means that it is still more of a hassle to use `lxc` in a way
  that would be required by this project.

* [`binctr`][binctr] was a proof-of-concept written by [Jessie
  Frazelle][jessfraz] and was the inspiration for the upstream rootless
  container work in runc. It has the interesting additional feature of creating
  a single static binary that contains both the container runtime and the root
  filesystem of the container.

* [`singularity`][singularity] was developed in parallel to the `runc`
  implementation of rootless containers. It is a runtime that was developed for
  HPC requirements, and so has quite a few odd design desisions based on those
  requirements. However, one point of note is that in its default configuration
  it has several `setuid` binaries that it uses for core operations (such as
  mounting the loopback device used as the rootfs), which make it impractical
  for the use-cases rootless containers were meant to solve. It appears that
  `singularity` does support user namespaces, but it's not clear whether a
  fully unprivileged setup is commonly used or has enough features to contend
  with `runc`'s rootless containers.

[bubblewrap]: https://github.com/projectatomic/bubblewrap
[bwrap-oci]: https://github.com/projectatomic/bwrap-oci
[lxc]: https://linuxcontainers.org/lxc/
[binctr]: https://github.com/jessfraz/binctr
[jessfraz]: https://blog.jessfraz.com/
[singularity]: https://www.sylabs.io/

## FAQ ##
<!-- If there is a question that hasn't been answered here, please open a PR! -->

### Who are we? ###

At the moment, a party of one. But if this sounds cool to you, feel free to
join me! I'm on [Mastodon](https://mastodon.social/@cyphar), and can be reached
at ([cyphar@cyphar.com](mailto:cyphar@cyphar.com)).
