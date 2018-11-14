# K8s Tools and Command

#### Kube-ps1

This tool use for display the current cluster.

Edit the `.bashrc`, `.bash_profile`, `.zshrc` and add the below code

```text
source /usr/local/Cellar/kube-ps1/0.6.0/share/kube-ps1.sh
export PROMPT='$(kube_ps1)'$PROMPT
```



