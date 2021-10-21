# rancher2.0 环境搭建

## 环境要求


1.  操作系统选择 centos7

2.  需要先安装docker

## 环境搭建


>  系统环境配置

```shell

sudo docker stop $(docker ps -a -q) //  stop停止所有容器

sudo docker  rm $(docker ps -a -q) //   remove删除所有容器

sudo systemctl stop firewalld && systemctl disable firewalld  //关闭防火墙

//关闭 SELinux

sudo setenforce 0

sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

sudo swapoff -a //关闭swap

//重启docker

sudo systemctl daemon-reload

sudo systemctl restart docker

```

>  安装 rancher

```shell

sudo docker run -d --restart=unless-stopped -p 8080:80 -p 1443:443 -v /home/wike/rancher:/var/lib/rancher/ rancher/rancher:stable

```


##  搭建集群

1. 首先需要创建集群

> 使用自定义模式，默认配置即可  

![image](https://csdn.52wike.com/2020-10-19/6fa0c602-d1a4-4c9a-b493-828137df6e74.png)

2. 添加主节点和工作节点


###  主节点

>  主节点需要勾选 etcd 和 Control

![image](https://csdn.52wike.com/2020-10-19/0202325b-8057-4022-b0f5-f3602ccfd381.png)

然后复制脚本到主节点主机执行

### 工作节点

> 工作节点不需要勾选 etcd 和 Control


![image](https://csdn.52wike.com/2020-10-19/1ff6fa18-7b1e-49d2-bff6-fa4c2618d16a.png)

然后复制脚本到主节点主机执行


###  注意事项

1. etcd端口不能被占用

2. 第一次安装节点会启动不了，报etcd错误，等所有节点都安装完成后，删除集群，重新添加集群，继续上述操作就可以正确搭建集群。