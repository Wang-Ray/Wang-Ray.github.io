---
layout: post
title: "实践Kubernetes"
date: 2018-07-19 09:58:13 +0800
categories: 云计算
tags: cloud Kubernetes container-orchestration
---

[Kubernetes](https://kubernetes.io/) is an open-source system for automating deployment, scaling, and management of containerized applications.

## 安装

版本

CentOS 7.6

docker-ce 19.03.9-3.el7

Kubernetes 1.19.16-0

calico 3.20



关闭防火墙

systemctl disable firewalld

systemctl stop firewalld

关闭swap

swapoff -a

vi /etc/fstab，删除swap挂载



修改主机名

```
hostnamectl set-hostname k8s-master
```

/etc/hosts



```
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3
sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
# Step 4: 更新并安装Docker-CE
sudo yum makecache fast
# Step 5：安装指定版本的docker-ce
yum -y install docker-ce-19.03.9-3.el7
```



```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```



yum install -y kubelet-1.19.16-0 kubeadm-1.19.16-0 kubectl-1.19.16-0



echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables



cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF



rm -rf /var/lib/etcd/



kubeadm init phase preflight



kubeadm init



kubeadm config print init-defaults

kubeadm config print join-defaults

kubeadm config images list

kubeadm config images pull



kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers

kubeadm init --image-repository registry.aliyuncs.com/google_containers



export KUBECONFIG=/etc/kubernetes/admin.conf



kubeadm join 192.168.1.231:6443 --token ot1cwn.jcjdue03ud93or6r \
    --discovery-token-ca-cert-hash sha256:81abfd322b69dcccef74381214fa54f52205dfca34a1a0bb8e253d805a433926



kubectl -n kube-system get configmap

kubectl get nodes



安装CNI网络插件

kubectl apply -f "https://docs.projectcalico.org/archive/v3.20/manifests/calico.yaml" 



kubectl get pods --all-namespaces







kubectl apply -f mysql-deploy.yaml

kubectl get deploy

kubectl get pods

docker ps |grep mysql



kubectl create -f mysql-svc.yaml

kubectl get svc mysql



注意修改mysql的地址

kubectl apply -f myweb-deploy.yaml

kubectl get svc myweb



http://192.168.1.232:30001/demo/



## 相关书籍

《Kubernetes权威指南：从Docker到Kubernetes实践全接触》

《Kubernetes权威指南——企业级容器云实战》

## 相关网站

[Kubernetes中文社区](https://www.kubernetes.org.cn)