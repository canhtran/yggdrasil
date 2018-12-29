---
description: Multiple nodes for K8s
---

# Centos 7

## Vagrant preparation

1. Box: centos/7
2. Memory = 2048
3. Docker

**Vagrantfile**

```perl
Vagrant.configure("2") do |config|
  config.vm.provision "docker"
  config.vm.provision "shell", path: "k8s_.sh"
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

**k8s\_setup.sh**

```bash
# disable SELINUX
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

# Turn of swap
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

## Install Kubernetes cluster

#### On master machine

```bash
$ kubeadm init --apiserver-advertise-address=192.168.99.100 --apiserver-cert-extra-sans=192.168.99.100 --node-name $(hostname -s) --pod-network-cidr 10.32.0.0/12
```

Set up for non root user

```bash
$ sudo --user=vagrant mkdir -p /home/vagrant/.kube
$ cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
$ chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config
```

Installing a pod network add-on

```bash
$ export KUBECONFIG=~vagrant/.kube/config
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.32.0.0/12"
```

#### On node machine

```
$ kubeadm join 192.168.99.100:6443 --token <token> --discovery-token-ca-cert-hash <ca-hash>
```



#### Note from Github commend about the issues

In our own Vagrant based devbox setup, we've worked around a couple of quirks \(which includes this issue\) by:

Add `--pod-network-cidr 10.32.0.0/12` to the `kubeadm init` command and use `https://cloud.weave.works/k8s/net?k8s-version=v1.11.1&env.IPALLOC_RANGE=10.32.0.0/12` as the URL for fetching the `weave.yaml`.

That'll fix the issue that the API server isn't reachable because the kube-proxy cannot distinguish between the pod IPs and the service IPs.

The next issue you'll run into with a Vagrant setup is probably that your nodes all report 10.0.2.15 as internal IP to the API server \(do a `kubectl describe node` and look at the output\). That can be fixed by choosing predictable worker IPs \(for example starting with 192.168.100.10\) and adding that as an extra argument to the kubelet's systemd configuration. For us Saltstack creates `/etc/systemd/system/kubelet.service.d/20-kubelet-node-ip.conf` and sets

```text
[Service]
        Environment="KUBELET_EXTRA_ARGS=--node-ip=192.168.100.10"
```

as the content. A reload of the unit is required but it depends on what and when the configuration is created.

It took us a lot of time to debug these issues but we finally have a stable and reproducible multi node Vagrant cluster.

A final note about debugging the Weave pods. We saved the weave.yaml file instead of directly applying it and replaced the `livenessProbe` with a `readinessProbe` in the `weave-net` DaemonSet to stop K8s from constantly reaping the Weave pods. It doesn't change the outcome \(node wasn't ready before and it's not after\) but it makes debugging a lot easier.

