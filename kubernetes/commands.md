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

Delete multiple pods

```text
for p in $(kubectl get pods -n <namespace> | grep <pod-status> | awk '{print $1}'); do kubectl delete pod $p -n <namespace> --grace-period=0 --force;done
```

Create namespace

```text
$ kubectl create namespace <namspace>
```

Get PersistentVolume / PersistentVolumeClaim / StorageClass

```bash
# persistentVolume
$ kubectl get pv
# persistentVolumeClaim
$ kubectl get pvc -n <namespace>
# storageClass
$ kubectl get sc
```

Get the cluster roles

```bash
$ kubectl get clusterroles
```

Create new user 

```bash
$ kubectl create clusterrolebinding jane --clusterrole=edit --user=jane
```

Get context

```text
$ kubectl config get-contexts
```

Create new context

```bash
$ kubectl config set-context <new-context-name> --cluster <cluster-from-old-context> --user <auth-info-from-old-context>
```

Delete the context

```text
$ kubectl config delete-context <old-context-name>
```

Label the node

```text
$ kubectl label node hostname node-role.kubernetes.io/worker=worker
```

Expose deployment to services

```text
$ kubectl expose deployment <deployment-name> --type=LoadBalancer --port=<port>
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

