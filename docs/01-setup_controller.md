# Setup Controller

Swapoff

```
sudo swapoff -a
```

Edit hosts to resolve other nodes

```
vagrant@vm01:~$ cat /etc/hosts
...

172.16.246.130 vm01 vm01.local
172.16.246.131 vm02
172.16.246.132 vm03
```

become root

```
sudo -i
```

Enable IPv4 forwarding

```
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# apply sysctl params without reboot
sysctl --system
```

Install docker

https://docs.docker.com/engine/install/ubuntu/


Enable cri plugin

```
sed -i 's/disabled_plugins = \[\"cri\"\]/\#disabled_plugins \= \[\"cri\"\]/g'  /etc/containerd/config.toml
systemctl restart containerd
```

Install kubeadm,kubectl,kubelet

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl

Start Controller

```
root@vm01:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:a0:12:a6 brd ff:ff:ff:ff:ff:ff
    inet 172.16.246.130/24 brd 172.16.246.255 scope global dynamic eth0
       valid_lft 1029sec preferred_lft 1029sec
    inet6 fe80::20c:29ff:fea0:12a6/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:61:ef:fc:46 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
``

```
# Use eth0 ip for apiserver-advertise-address

```
root@vm01:~# kubeadm init --pod-network-cidr 10.244.0.0/16 --apiserver-advertise-address 172.16.246.130
```

Result would be like below:

```

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.246.130:6443 --token p310q3.no9wg6s99d4s0qb9 \
	--discovery-token-ca-cert-hash sha256:a32e9afa3bb6a013f047e5f423b8ffa8f600836e31d60e69da180795dd545cc3
```

export ENV for kubectl

```
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >>  ~/.bashrc
echo "alias k='kubectl'" >>  ~/.bashrc
source ~/.bashrc
```

test:

```
root@vm01:~# k get no
NAME   STATUS     ROLES           AGE     VERSION
vm01   NotReady   control-plane   2m26s   v1.27.4
root@vm01:~# k get po -A
NAMESPACE     NAME                           READY   STATUS    RESTARTS   AGE
kube-system   coredns-5d78c9869d-rj24b       0/1     Pending   0          2m25s
kube-system   coredns-5d78c9869d-vrgl7       0/1     Pending   0          2m25s
kube-system   etcd-vm01                      1/1     Running   0          2m38s
kube-system   kube-apiserver-vm01            1/1     Running   0          2m38s
kube-system   kube-controller-manager-vm01   1/1     Running   0          2m38s
kube-system   kube-proxy-ngt85               1/1     Running   0          2m25s
kube-system   kube-scheduler-vm01            1/1     Running   0          2m38s
```

Apply flannel following https://github.com/flannel-io/flannel#deploying-flannel-manually :

```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

check node/pod is ready:

```
root@vm01:~# k get no
NAME   STATUS   ROLES           AGE    VERSION
vm01   Ready    control-plane   5m6s   v1.27.4

root@vm01:~# k get po -A
NAMESPACE      NAME                           READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-kdlq5          1/1     Running   0          71s
kube-system    coredns-5d78c9869d-rj24b       1/1     Running   0          5m25s
kube-system    coredns-5d78c9869d-vrgl7       1/1     Running   0          5m25s
kube-system    etcd-vm01                      1/1     Running   0          5m38s
kube-system    kube-apiserver-vm01            1/1     Running   0          5m38s
kube-system    kube-controller-manager-vm01   1/1     Running   0          5m38s
kube-system    kube-proxy-ngt85               1/1     Running   0          5m25s
kube-system    kube-scheduler-vm01            1/1     Running   0          5m38s
```
