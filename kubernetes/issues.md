---
description: Issues related to k8s
---

# Issues

**\#1 Node in NotReady mode due to docker service cannot start.**

```text
# Delete /etc/docker/daemon.json
$ rm -rf /etc/docker/daemon.json
$ service restart docker
```

