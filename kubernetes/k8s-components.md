# k8s components

## SC / PV / PVC

### Create Persistent Volume

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: hub-db-dir
  labels:
    type: local
spec:
  persistentVolumeReclaimPolicy: Recycle
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/jhub"
```

### Create Persistent Volume Claim

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: hub-db-dir
spec:
  storageClassName: slow
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

### Create Storage Class

[https://raw.githubusercontent.com/kubernetes/kubernetes/release-1.13/cluster/addons/storage-class/local/default.yaml](https://raw.githubusercontent.com/kubernetes/kubernetes/release-1.13/cluster/addons/storage-class/local/default.yaml)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  namespace: kube-system
  name: standard
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  labels:
    addonmanager.kubernetes.io/mode: EnsureExists
provisioner: kubernetes.io/host-path
```

## Nginx

{% embed url="https://kubernetes.github.io/ingress-nginx/deploy/\#using-helm" %}

Config file for Nodeport

```yaml
controller:
  service:
    type: NodePort
    nodePorts:
      http: 30000
  replicaCount: 1
```

