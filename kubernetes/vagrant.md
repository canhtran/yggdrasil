# Vagrant

Using [Kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/) to install kubernetes cluster on vagrant

Vagrant preparation

1. bento/ubuntu-16.04
2. private network as dhcp
3. gui = false and memory = 2048
4. swap off
5. docker installed

Vagrantfile

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

Install kubernetes

```text
$ apt-get update && apt-get install -y apt-transport-https curl
$ $ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```

```text
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

```text
$ apt-get update
$ apt-get install -y kubelet kubeadm kubectl
$ apt-get hold kubelet kubeadm kubectl
```

Config

