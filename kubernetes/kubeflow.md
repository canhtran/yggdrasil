# Kubeflow

## Install kubeflow on ubuntu with kubeadm

- Create new namespace: kubeflow
- Create a directory where you want to download the source to

```
mkdir ${KUBEFLOW_SRC}
cd ${KUBEFLOW_SRC}
export KUBEFLOW_TAG=v0.3.0
curl https://raw.githubusercontent.com/kubeflow/kubeflow/${KUBEFLOW_TAG}/scripts/download.sh | bash
```

Deploy Kubeflow

- Create a directory to store the configs

```
${KUBEFLOW_REPO}/scripts/kfctl.sh init ${KFAPP} --platform none
cd ${KFAPP}
${KUBEFLOW_REPO}/scripts/kfctl.sh generate k8s
${KUBEFLOW_REPO}/scripts/kfctl.sh apply k8s
```