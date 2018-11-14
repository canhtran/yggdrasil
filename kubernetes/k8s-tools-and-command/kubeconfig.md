# Kubeconfig

#### Set up kubeconfig

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



