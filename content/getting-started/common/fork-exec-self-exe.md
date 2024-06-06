---
title: "Operation not permitted fork/exec /proc/self/exe"
weight: 40
---

AppArmor is a Linux security module that restricts programs' capabilities by enforcing access controls defined in profiles. 
It provides an additional layer of security by limiting what resources applications can access.

Based on <https://ubuntu.com/blog/ubuntu-23-10-restricted-unprivileged-user-namespaces>

After running `containerd-rootless-setuptool.sh check` or `containerd-rootless-setuptool.sh install`

If you get the error below

```
[rootlesskit:parent] error: failed to start the child: fork/exec /proc/self/exe: operation not permitted
```

Try to run `rootlesskit bash`, it will generate a script, based on hint from
<https://github.com/rootless-containers/rootlesskit/blob/master/pkg/parent/warn.go>

```
ubuntu@energetic-anemone:~$ rootlesskit bash
WARN[0000] [rootlesskit:parent] This error might have happened because /proc/sys/kernel/apparmor_restrict_unprivileged_userns is set to 1  error="fork/exec /proc/self/exe: permission denied"
WARN[0000] [rootlesskit:parent] Hint: try running the following commands:


########## BEGIN ##########
cat <<EOT | sudo tee "/etc/apparmor.d/home.ubuntu.bin.rootlesskit"
# ref: https://ubuntu.com/blog/ubuntu-23-10-restricted-unprivileged-user-namespaces
abi <abi/4.0>,
include <tunables/global>

/home/ubuntu/bin/rootlesskit flags=(unconfined) {
  userns,

  # Site-specific additions and overrides. See local/README for details.
  include if exists <local/home.ubuntu.bin.rootlesskit>
}
EOT
sudo systemctl restart apparmor.service
########## END ##########

[rootlesskit:parent] error: failed to start the child: fork/exec /proc/self/exe: permission denied

```

more context: <https://github.com/rootless-containers/rootlesskit/issues/434>
