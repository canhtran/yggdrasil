---
description: 'https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/'
---

# Kubeadm

## Vagrant preparation

Change box to `bento/ubuntu-16.04` for Ubuntu version

{% code-tabs %}
{% code-tabs-item title="Centos" %}
```perl
 Vagrant.configure("2") do |config|
  config.vm.provision "docker"
  config.vm.provision "shell", path: "install_k8s.sh"
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "2048"
    vb.cpus= 2
    vb.customize ["modifyvm", :id, "--cableconnected1", "on"]
  end

  config.vm.define "k8s1" do |k8s1|
    k8s1.vm.box = "centos/7"
    k8s1.vm.hostname = 'k8s1'
    k8s1.vm.network "private_network", ip: "192.168.99.100"
  end

  config.vm.define "k8s2" do |k8s2|
    k8s2.vm.box = "centos/7"
    k8s2.vm.hostname = 'k8s2'
    k8s2.vm.network "private_network", ip: "192.168.99.101"
  end

  config.vm.define "k8s3" do |k8s3|
    k8s3.vm.box = "centos/7"
    k8s3.vm.hostname = "k8s3"
    k8s3.vm.network "private_network", ip: "192.168.99.102"
  end
end
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## install\_k8s.sh

{% code-tabs %}
{% code-tabs-item title="Centos" %}
```bash
# disable SELINUX
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

# Turn off swap
swapoff -a
sed -i.bak -r 's/(.+ swap .+)/#\1/' /etc/fstab

# enable br_netfilter
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables

# Download kubernetes library
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

# install k8s packages
yum install -y kubelet-1.13.0 kubeadm-1.13.0 kubectl-1.13.0 --disableexcludes=kubernetes

# start kubelet
systemctl enable kubelet && systemctl start kubelet

# br_netfilter
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# Config cgroup driver
sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl daemon-reload
systemctl restart kubelet
```
{% endcode-tabs-item %}

{% code-tabs-item title="Ubuntu" %}
```perl
# disable SELINUX
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

# turn off swap
swapoff -a
sed -i.bak -r 's/(.+ swap .+)/#\1/' /etc/fstab

# add kubernetes to deb
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

# install packages
apt-get update
apt-get install -y kubelet==1.13.0 kubeadm==1.13.0 kubectl==1.13.0
apt-get hold kubelet kubeadm kubectl

# cgroup driver
sed -i '0,/ExecStart=/s//Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"\n&/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Install kubernetes cluster

### On master machine

```bash
kubeadm init --apiserver-advertise-address=192.168.99.100 --apiserver-cert-extra-sans=192.168.99.100 --node-name $(hostname -s) --kubernetes-version=1.13.0 --pod-network-cidr=10.244.0.0/16 --apiserver-bind-port=443
```

Set up for non root user

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Installing a pod network add-on

```bash
export KUBECONFIG=~vagrant/.kube/config
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```

#### Create service account for helm \(optional\)

```bash
cat > tiller-serviceaccount.yaml << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: tiller-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: tiller
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: ""
EOF

kubectl create -f tiller-serviceaccount.yaml
```

### Install helm tiller

```text
helm init --service-account tiller --upgrade
```

### Join node machine

```
kubeadm join 192.168.99.100:6443 --token <token> --discovery-token-ca-cert-hash <ca-hash>
```

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

