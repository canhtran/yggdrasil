# Helm Chart

Must have full control of `kubectl`

Depend on the version of os, download helm from: [https://github.com/helm/helm/releases](https://github.com/helm/helm/releases)

Copy the `helm` and put in `usr/local/bin`

Or [https://docs.helm.sh/using_helm/#from-script](https://docs.helm.sh/using_helm/#from-script)

```
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

### Getting started

Init helm and install Tiller (Tiller will be figured by reading Kubernetes configuration file `$HOME/.kube/config`)

```
$ helm init
```

To install helm in a specific namespace.

```
export TILLER_NAMESPACE=data
```

The successul of installation will look like

```
→ kubectl get pods -n data                                                                                                              │        http://localhost:8888/?token=736e4884e9c96d8f0993ec79964b63528420972d1aacf4ba
NAME                             READY     STATUS    RESTARTS   AGE                                                                     │[I 11:09:11.891 NotebookApp] Accepting one-time-token-authenticated connection from ::1
tiller-deploy-6b8ddbdd5b-zrbf9   1/1       Running   0          1d
```

### Basic commands

Create helm package

```
$ helm create helm-chart
```

Render helm template

```
$ helm template -n titanic ./helm-chart | less
```

Install helm package

```
$ helm install --name titanic ./helm-chart  --namespace data
```

Install helm package with specific `values.yml`

```
$ helm install -f values-file.yaml titanicship .
```

Delete helm package

```
$ helm delete titanic --purge
```

Update helm package

```
$ helm upgrade titanic ./helm-chart
```

Check helm version

```
$ helm version --kube-context kubernetes-staging-1a-dex --tiller-namespace=data
```


### Issue

Some time I encounterd a problem related to priviledges

```
Error: release nginx failed: namespaces "default" is forbidden: User "system:serviceaccount:kube-system:default" cannot get namespaces in the namespace "default"
```

That because I don't have permission to deploy tiller, add account for it will solve the problem

```
$ kubectl create serviceaccount --namespace kube-system tiller
serviceaccount "tiller" created

$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
clusterrolebinding "tiller-cluster-rule" created

$ kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' 
deployment "tiller-deploy" patched
```