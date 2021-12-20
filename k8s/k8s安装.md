# k8s 1.20安装 （无docker）

## 操作

1 关闭防火墙
```
systemctl status firewalld.service 
systemctl stop firewalld.service 
systemctl disable firewalld.service 
```

2 关闭swap
```
swapoff -a
vi /etc/fstab  //删除 /mnt/swap swap swap defaults 0 0 这一行或者注释掉这一行

```

3 关闭SELINUX
```
setenforce 0 
vi /etc/selinux/config  //将SELINUX=enforcing改为SELINUX=disabled

```

4 安装必要工具
```
 yum install -y yum-utils 
 yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo 
 yum install containerd -y
 cat <<EOF > /etc/yum.repos.d/kubernetes.repo
 [kubernetes]
 name=Kubernetes
 baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
 enabled=1
 gpgcheck=1
 repo_gpgcheck=1
 gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
         https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
 EOF
 yum makecache
 yum -y install kubelet-1.20.2  kubeadm-1.20.2  kubectl-1.20.2


```

5 修改主机名

> 因为k8s每个节点hostname不能一样所以需要修改主机名

```

hostnamectl set-hostname k8s-xxx

vi /etc/hosts //使每个节点能通过hostname ping通

```

6 修改配置
```
wget https://wikecloud.oss-cn-hangzhou.aliyuncs.com/k8s/config.toml
cp config.toml /etc/containerd/config.toml
echo 1 > /proc/sys/net/ipv4/ip_forward
modprobe br_netfilter
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

cat > /etc/sysconfig/kubelet << EOF
KUBELET_EXTRA_ARGS=--cgroup-driver=systemd
EOF
```

7 启动程序
```
systemctl daemon-reload && systemctl start containerd
systemctl enable containerd

systemctl enable kubelet

systemctl daemon-reload  
systemctl start containerd 
systemctl restart kubelet 
```

8 初始化集群
```
kubeadm init --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers  --kubernetes-version=1.20.2 --cluster-cidr=10.244.0.0/16 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12  --cri-socket=unix:///run/containerd/containerd.sock

```

9 安装 crictl

```
wget https://wikecloud.oss-cn-hangzhou.aliyuncs.com/k8s/crictl
chmox +x crictl
mv ./crictl /usr/local/bin
```

10 安装网络插件

```
wget https://wikecloud.oss-cn-hangzhou.aliyuncs.com/k8s/flanneld-v0.13.1-rc1-amd64.docker

ctr -n k8s.io image import flanneld-v0.13.1-rc1-amd64.docker

wget https://wikecloud.oss-cn-hangzhou.aliyuncs.com/k8s/flannel.txt

kubectl apply -f  flannel.txt 

```

11 安装helm

```
wget https://wikecloud.oss-cn-hangzhou.aliyuncs.com/k8s/helm
chmod +x helm
mv ./helm /usr/local/bin

```

12 安装nginx-ingress

```
wget https://wikecloud.oss-cn-hangzhou.aliyuncs.com/k8s/ingress-nginx.tar.gz
helm install nginx-ingress  ingress-nginx.tar.gz

```

13 去除master污点 （可选）

```
kubectl taint nodes --all node-role.kubernetes.io/master

```