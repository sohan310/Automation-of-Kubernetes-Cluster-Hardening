
# Automation of Kubernetes Cluster Hardening

This is a small project to make a Kubernetes Cluster more secure by applying rules to the files of cluster.

This project is using Centos 7 as base platform to run and configure the K8s cluster.


## Prerequisite

- Centos 7
- 16 GB RAM
- 100 GB Hard Disk Space
- 8 CPUs




## Installation

### Master Node
- Centos 7
- 8 GB RAM
- 80 GB Hard Disk Space
- 4 CPUs 

### Worker Node
- Centos 7
- 2 GB RAM
- 20 GB Hard Disk Space
- 2 CPUs

## Firstly creation of Master and Worker Node
Install the Centos 7 on both master and worker machine. After completion of Installation. Run the below command to configure the network connection between master and worker.
```bash
  nano /etc/sysconfig/network-scripts/ifcfg-ens33
```
### Now configure in master and worker according to the below description.
```bash
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
IPV6_AUTOCONF=no
IPV6_DEFROUTE=no
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=9b899949-fdbd-40a9-857a-xxxxxxxxxxxx
DEVICE=ens33
ONBOOT=yes
IPADDR=xxx.xxx.xxx.xxx
PREFIX=24
GATEWAY=xxx.xxx.xxx.xxx
DNS1=xxx.xxx.xxx.xxx
DNS2=8.8.8.8
DNS3=8.8.4.4
```
### Once you configure run the below command.
 ```bash
 systemctl restart network
 ```

 Now add a hostname to both the machines
 ```bash
 hostnamectl set-hostname <hostname>
 ```
 Now after giving hostnames to machines make an entry in the hosts files to make a successful ping.

 Once pinging is done run the command.
 ```bash
 yum update -y
 ```
Now your machines are updated with new packages.
## Run the script of kubernetes Installation
The script starts with disabling the SELinux.
- The reason to disable the SELinux is to easily access host filesystem in all containers.
```bash
sed -i 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
```
Now change the setenforce to 0 so that it will temporarily disable SELinux
```bash
setenforce 0
```
Disable the swap partition (It is mandatory)
```bash
# First diasbale swap
swapoff -a

# And then to disable swap on startup in /etc/fstab
sed -i '/swap/d' /etc/fstab
```
Now enable the net_filter module
```bash
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-ip6tables
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
touch /etc/sysctl.d/k8s.conf
echo 'net.bridge.bridge-nf-call-ip6tables = 1' > /etc/sysctl.d/k8s.conf
echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.d/k8s.conf
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p

```
Uptil now we have configure the machines before installing kubernetes, if we miss any of this steps our cluster will not work properly.
We have to apply this on both the nodes.


## Now we are going to install docker and container packages.
before installing docker install the utilities packages required for docker.
```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```
Then run the below commands.
```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io
```
Now enable the services.
```bash
systemctl start --now docker
systemctl enable --now docker
```
Create a kubernetes repository.
```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
Now install the kubernetes packages
```bash
# Install kubectl, kubelet and kubeadm
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```
enable the services
```bash
# Start and enable kubelet service
systemctl start --now kubelet
systemctl enable --now kubelet
```
## Uptil now we have install all the packages we need, all the above steps need to be reapeated on worker node.

### Now the following below steps will only done on master node.
```bash
kubeadm init --pod-network-cidr 10.244.0.0/16 --apiserver-advertise-address=xxx.xxx.xxx.xxx
```
Once the above command runs it takes few minutes to pull the images from kubernetes and a token will created for worker node.

```bash
kubeadm join xxx.xxx.xxx.xxx:6443 --token ou2z10.55rrc31a2j9rl5tp --discovery-token-ca-cert-hash sha256:801968fa1a3ec3a71e9948c85178346a0056195f6072681a6cbd64960cdfcded
```
Now copy the token and paste it in a file of worker node and run the file like this
```bash
./join-token
```
Now create a folder for kubernetes and give the excutiable permissions
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Once the folder is created run the below command to create a pod network
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
Now the pod network is created between master and worker.

### To check the statud of cluster we created 
```bash
kubectl get nodes
kubectl get pods --all-namespaces
```
## This conclude the Installation and configuration of kubernetes