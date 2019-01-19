# Commands

## kubectl

Get pods

```text
$ kubectl get pod
$ kubectl get pod --show-labels -o wide
```

Get config

```text
$ kubectl config get-contexts
```

Get deployment

```text
$ kubectl get deployment -n <namespace>
```

Get service

```text
$ kubectl get svc -n <namespace>
```

Port forwarding

```text
$ kubectl port-forward <pod-name> 5000:5000 -n data
```

Access to pod

```text
$ kubectl exec -it <pod-name> bash
```

Get log of pod

```text
$ kubectl logs <pod-name>
```

Delete pod

```text
$ kubectl delete <pod-name>
```

Create namespace

```text
$ kubectl create namespace <namspace>
```

Get PersistentVolume / PersistentVolumeClaim / StorageClass

```bash
# persistenceVolume
$ kubectl get pv
# persistenceVolumeClaim
$ kubectl get pvc -n <namespace>
# storageClass
$ kubectl get sc
```

## kubectx

List all the kubeconfigs

```text
$ kubectx
```

Select the kubeconfig

```text
$ kubectx <kubeconfig name>
```

