# K8s Tools

## Libraries

* Helm \([https://docs.helm.sh/](https://docs.helm.sh/)\)
* Kubectl \([https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)\)
* Kubectx \([https://github.com/ahmetb/kubectx](https://github.com/ahmetb/kubectx)\)
* Kube-ps1 \(to display kubernetes cluster\) \([https://github.com/jonmosco/kube-ps1](https://github.com/jonmosco/kube-ps1)\)

## Kubeconfig

Create new directory

`$ mkdir ~/.kubeconfigs`

Copy and put all configs file into the directory

Edit the `.bashrc`, `.bash_profile`, `.zshrc` and add the below code

```text
KUBECONFIG=""
for i in $HOME/.kubeconfigs/*; do
  KUBECONFIG="$KUBECONFIG:${i}"
done
export KUBECONFIG
```

## Kube-ps1 \(Optional\)

This tool use for display the current cluster.

Edit the `.bashrc`, `.bash_profile`, `.zshrc` and add the below code

```text
source /usr/local/Cellar/kube-ps1/0.6.0/share/kube-ps1.sh
export PROMPT='$(kube_ps1)'$PROMPT
```

