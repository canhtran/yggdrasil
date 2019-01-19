---
description: Issues and Solution
---

# Jupyterhub in kubeadm cluster

## Issue \#1: hub service is pending

**Reason**: the `persistanceVolumeClaim` cannot claim the volume.

**Solution**: create a `persistentVolume` name `hub-db-dir` before running the helm command.

## Issue \#2: hub service stucks at `ContainerCreating`

**Reason:** permissions denied on writing to mount volume, by default when create the hostpath, kubernetes will add the user and group as root.

```bash
[E 2018-12-05 15:36:39.594 JupyterHub app:1952]
    Traceback (most recent call last):
      File "/usr/local/lib/python3.6/dist-packages/jupyterhub/app.py", line 1949, in launch_instance_async
        await self.initialize(argv)
      File "/usr/local/lib/python3.6/dist-packages/jupyterhub/app.py", line 1673, in initialize
        self.init_secrets()
      File "/usr/local/lib/python3.6/dist-packages/jupyterhub/app.py", line 1055, in init_secrets
        with open(secret_file, 'w') as f:
    PermissionError: [Errno 13] Permission denied: '/srv/jupyterhub/jupyterhub_cookie_secret'
```

**Solution**: change the uid and fsGid to 0 in `config.yaml` file when running helm.

```yaml
hub:
  uid: 0
  fsGid: 0
```

