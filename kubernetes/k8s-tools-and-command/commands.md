# Commands


### kubectl

Get pods

```
$ kubectl get pod
$ kubectl get pod --show-labels -o wide
```

Get config

```
$ kubectl config get-contexts
```

Get deployment

```
$ kubectl get deployment -n <namespace>
```

Get service

```
$ kubectl get svc -n <namespace>
```

Port forwarding

```
$ kubectl port-forward <pod-name> 5000:5000 -n data 
```

Access to pod

```
$ kubectl exec -it <pod-name> bash
```

Get log of pod

```
$ kubectl logs <pod-name>
```

Delete pod

```
$ kubectl delete <pod-name>
```

Create namespace

```
$ kubect create namespace <namspace>
```

### kubectx

List all the kubeconfigs

```
$ kubectx
```

Select the kubeconfig

```
$ kubectx <kubeconfig name>
```