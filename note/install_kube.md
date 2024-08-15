
# Install a kubernetes cluster using kubeadm

## environment
- CentOS 7
- kubeadm

## Steps
### 0. Prepare the Kubernetes environment
1. yum update 
	1. centos mirrors may need to be update
2. [[202408151539-Install docker on CentOS|Install docker on CentOS]]
3. install [[202408151539-containerd|containerd]] (included in step 2)
4. stop & disable selinux, swap, firewalld

```bash
# using root account
su
# disable selinux
setenforce 0
gedit /etc/selinux/config # make selinux=disable
sestatus
# disable swap
swapoff -a	
gedit /etc/fstab # either comment out the swap lines,
sed -i '/\tswap\t/d' /etc/fstab # or just run this line...
# disable firewalld
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
# reboot
reboot
```

5. check network: 
	1. port 6443 (if open: but firewalld is disabled in step 4, so it does not matter); 
	2. hostname setup: `\etc\hosts`
	4. ip -> static: `ifconfig` -> `gedit \etc\sysconfig\network-scripts\[script_name]` -> `boot...type=none; ip_addr=...; ...`
6. [[202408151543-install kubeadm, kubelet, kubectl|install kubeadm, kubelet, kubectl]]
7. enable and start docker, kubelet

### 1. Init the master node
1. kubeadm init 

```bash
kubeadm init --pod-network-cidr=[ip_addr]]/24 --apiserver-advertise-address=[ip_addr] --ignore-preflight-errors=all 
```

2. cp & run the given commands

### 2. Join the worker nodes
Use the join command given by the master node to join the worker node
1. run `kubeadm init` first to make things prepared... (weird, seems like it needs the init to prepare some config or cert files for the kubelet)
2. `kubeadm reset`: delete the cluster info we just created
3. run the join command
Join command
```bash
kubeadm join <api-server-ip:port> --token <token-value> \  
--discovery-token-ca-cert-hash sha256:<hash value>
```

to get the token, in the master node:
```bash
kubeadm token create --print-join-command
```

to fix some join error: ipv4_forwarding is not set to 1...
```bash
modprobe br_netfilter
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo 1 > /proc/sys/net/ipv4/ip_forward
```
## Tutorials
- https://github.com/vevsatechnologies/Install-Kubernetes-on-CentOs/blob/master/master.sh 
