# docker环境搭建

## 环境要求


>  操作系统选择 centos7


## 环境搭建


>  安装 docker-ce

```shell
sudo yum install -y yum-utils \device-mapper-persistent-data \lvm2

sudo yum-config-manager \--add-repo \https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo

sudo yum makecache fast

sudo yum install docker-ce

sudo systemctl enable docker

sudo systemctl start docker

sudo usermod -aG docker $USER

newgrp - docker

sudo systemctl restart docker
```

>  安装 docker-compose

```shell

sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

```
