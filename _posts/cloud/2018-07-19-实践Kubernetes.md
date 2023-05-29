---
layout: post
title: "实践Kubernetes"
date: 2018-07-19 09:58:13 +0800
categories: 云计算
tags: cloud Kubernetes container-orchestration
---

[Kubernetes](https://kubernetes.io/) is an open-source system for automating deployment, scaling, and management of containerized applications.

## 概念

* master

* node

* pod

label

* service

label selector

* container

pause

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



建议在主机上禁用 SELinux （修改文件／etc/sysconfig/selinux, SELINUX=enforcing 修改为 SELIN口(=中sabled) 



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

mysql-deploy.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: mysql
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.7
        name: mysql
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "123456"
```



```
$ kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
mysql   1/1     1            1           24h
myweb   2/2     2            2           24h
```



```shell
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
mysql-596b96985c-p9l52   1/1     Running   0          23h
myweb-7d75cf577c-qzbl8   1/1     Running   0          23h
myweb-7d75cf577c-x9mgd   1/1     Running   0          23h
```



```shell
$ docker ps |grep mysql
b78afdeb29c5   mysql                                               "docker-entrypoint.s…"   24 hours ago   Up 24 hours             k8s_mysql_mysql-596b96985c-p9l52_default_077df04c-d430-49c9-8070-db9010e00ec6_0
227547a3c1e2   registry.aliyuncs.com/google_containers/pause:3.2   "/pause"                 24 hours ago   Up 24 hours             k8s_POD_mysql-596b96985c-p9l52_default_077df04c-d430-49c9-8070-db9010e00ec6_0
```



kubectl create -f mysql-svc.yaml

mysql-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
    - port: 3306
  selector:
    app: mysql
```

kubectl get svc mysql

```shell
$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          28h
mysql        ClusterIP   10.104.218.155   <none>        3306/TCP         23h
myweb        NodePort    10.106.209.221   <none>        8080:30001/TCP   23h
```



注意修改mysql的地址

kubectl apply -f myweb-deploy.yaml

myweb-deploy.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myweb
  name: myweb
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myweb
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
      - image: kubeguide/tomcat-app:v1
        name: myweb
        ports:
        - containerPort: 8080
        env:
        - name: MYSQL_SERVICE_HOST
          value: 10.104.218.155
```



kubectl create -f myweb-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myweb
spec:
  type: NodePort
  ports:
    - port: 8080
      nodePort: 30001
  selector:
    app: myweb
```



kubectl get deployments.apps

kubectl edit deployment/myweb -o yaml --save-config

kubectl get svc myweb



http://192.168.1.232:30001/demo/



## 相关书籍

《Kubernetes权威指南：从Docker到Kubernetes实践全接触》

《Kubernetes权威指南——企业级容器云实战》

## 相关网站

[Kubernetes中文社区](https://www.kubernetes.org.cn)