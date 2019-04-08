---
layout: post
title: "基于post receive git hook实现gitlab同步"
date: 2017-10-24 09:25:00 +0800
categories: 工具
tags: tool git gitlab
---

基于post receive git hook实现git仓库的实时同步，提交到原仓库的内容会同时同步给镜像仓库。

原仓库（gitlab）：http://192.168.70.244/CTS/config

镜像仓库（gitlab，最开始是一个`未经初始化的空仓库`）：http://192.168.70.139/root/config

## 实现原仓库用户基于git协议访问镜像仓库

登录原仓库gitlab主机

```shell
# 切换到git用户（gitlab的运行用户，根目录/var/opt/gitlab）
$ su - git
# 生产ssh密钥对，一路回车即可
$ ssh-kengen
$ more ~/.ssh/id_rsa.pub
# 将公钥加入到镜像仓库对应用户的SSH keys中
# 验证SSH key添加是否成功
$ ssh -T git@192.168.70.139
```

## 添加远程（镜像）仓库

```shell
# 进入原仓库根目录
$ cd /var/opt/gitlab/git-data/repositories/CTS/config.git/
# 添加远程（镜像）仓库
$ git remote add --mirror=push gitlab-70-139 git@192.168.70.139:root/config.git
```

## post receive git hook配置

```shell
# 在原仓库根目录新建custom_hooks
$ mkdir custom_hooks
$ cd custom_hooks
# 新建post-receive
$ touch post-receive
# 可执行权限
$ chmod -R 755 post-receive
# 编辑post-receive
$ vi post-receive
```

内容如下：

```shell
#!/bin/bash
echo "$USER"
exec git push -u gitlab-70-139 &
```

## 完成

经过以上配置后，当我们提交修改到原仓库时，代码也会同步提交到镜像仓库。

```shell
$ git push
对象计数中: 4, 完成.
Delta compression using up to 8 threads.
压缩对象中: 100% (4/4), 完成.
写入对象中: 100% (4/4), 351 bytes | 0 bytes/s, 完成.
Total 4 (delta 3), reused 0 (delta 0)
remote: 
remote: To 192.168.70.139:root/config.git
remote:    c825505..19ce9fa  master -> master
remote: Branch master set up to track remote branch master from gitlab-70-139.
To 192.168.70.244:CTS/config.git
   c825505..19ce9fa  master -> master
```

如果镜像仓库宕机了，提交代码提示如下，提交到原仓库成功，但是同步到镜像仓库失败，待镜像仓库恢复后，下次提交会全部同步：

```shell
$ git push
对象计数中: 4, 完成.
Delta compression using up to 8 threads.
压缩对象中: 100% (4/4), 完成.
写入对象中: 100% (4/4), 339 bytes | 0 bytes/s, 完成.
Total 4 (delta 3), reused 0 (delta 0)
remote: 
remote: GitLab: Failed to authorize your Git request: internal API unreachable
remote: fatal: Could not read from remote repository.
remote: 
remote: Please make sure you have the correct access rights
remote: and the repository exists.
To 192.168.70.244:CTS/config.git
   19ce9fa..6763f9a  master -> master
```

备注：参考[使用post receive hook同步Git仓库](http://www.itmuch.com/work/git-repo-sync-with-post-receive/)