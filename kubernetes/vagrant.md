# Vagrant

Using Kubeadm to install kubernetes cluster on vagrant. The guide for installing kubeadm can be found [here](https://kubernetes.io/docs/setup/independent/install-kubeadm/).

## Vagrant preparation

1. Box: bento/ubuntu-16.04
2. Private network as dhcp
3. Memory = 2048
4. Docker

**Vagrantfile**

```text
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.box_check_update = false
  config.vm.network "private_network", type: "dhcp"
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "2048"
    vb.customize ["modifyvm", :id, "--cableconnected1", "on"]
  end
  config.vm.provision "docker"
  config.vm.provision "shell", path: "kube-install.sh"
end
```

**Config box**

Turn off swap

```text
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

## Install Kubernetes cluster

**Install kubernetes**

```text
$ apt-get update && apt-get install -y apt-transport-https curl
$ $ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
$ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

```text
$ apt-get update
$ apt-get install -y kubelet kubeadm kubectl
$ apt-get hold kubelet kubeadm kubectl
```

**Config**

Configure cgroup driver used by kubelet on Master Node

```text
$ sed -i '0,/ExecStart=/s//Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"\n&/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

Get IP Address

```text
$ IPADDRESS=`ifconfig eth1 | grep "inet addr" | cut -d ':' -f 2 | cut -d ' ' -f 1`
```

Set up Kubeadm

```text
$ kubeadm init --apiserver-cert-extra-sans=$IPADDRESS  --node-name $(hostname -s)
```

Set up for non root user

```text
$ sudo --user=vagrant mkdir -p /home/vagrant/.kube
$ cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
$ chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config
```

Installing a pod network add-on

```text
$ export KUBECONFIG=~vagrant/.kube/config
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

