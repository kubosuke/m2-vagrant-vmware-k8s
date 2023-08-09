
# Setup Worker


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

Run join command 

- On controller (vm01)

```
kubeadm token create --print-join-command
```

- On worker (vm02 or vm03)

put in the output of above `kubeadm token create` command

```
kubeadm join 172.16.246.130:6443 --token giz4l5.eaehjb7u4kxuxi0p --discovery-token-ca-cert-hash sha256:a32e9afa3bb6a013f047e5f423b8ffa8f600836e31d60e69da180795dd545cc3
```

Check worker node status 

```
root@vm01:~# k get no
NAME   STATUS     ROLES           AGE   VERSION
vm01   Ready      control-plane   13m   v1.27.4
vm02   NotReady   <none>          4s    v1.27.4
```

Label vm02/03 as worker node

```
root@vm01:~# k label no vm02 node-role.kubernetes.io/worker=worker
node/vm02 labeled
root@vm01:~# k get no
NAME   STATUS   ROLES           AGE     VERSION
vm01   Ready    control-plane   19m     v1.27.4
vm02   Ready    worker          5m50s   v1.27.4
```