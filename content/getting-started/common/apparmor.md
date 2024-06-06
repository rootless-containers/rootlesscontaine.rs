---
title: "[Optional] AppArmor"
weight: 60
---

{{< hint info >}}
**Note**

Configuring AppArmor is needed only on Ubuntu 24.04 or later,
with RootlessKit installed under a non-standard path.
{{< /hint>}}

If you face an error like `[rootlesskit:parent] error: failed to start the child: fork/exec /proc/self/exe: operation not permitted`,
try running the following commands:
```bash
cat <<EOT | sudo tee "/etc/apparmor.d/usr.local.bin.rootlesskit"
abi <abi/4.0>,
include <tunables/global>

/usr/local/bin/rootlesskit flags=(unconfined) {
  userns,

  # Site-specific additions and overrides. See local/README for details.
  include if exists <local/usr.local.bin.rootlesskit>
}
EOT
sudo systemctl restart apparmor.service
```

The `/usr/local/bin/rootlesskit` string should be changed to the actual path of `rootlesskit`.

This step is *not* needed when `rootlesskit` is installed in the standard path (`/usr/bin/rootlesskit`).
