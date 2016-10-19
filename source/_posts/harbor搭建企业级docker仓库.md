title: harbor搭建企业级docker仓库
date: 2016-10-08 15:22:56
tags: [docker]
---



### 简介：

Harbor项目是帮助用户迅速搭建一个企业级的registry 服务。

它以Docker公司开源的registry为基础，提供了管理UI, 基于角色的访问控制(Role Based Access Control)，AD/LDAP集成、以及审计日志(Audit logging) 等企业用户需求的功能，同时还原生支持中文。



### 安装docker

```
https://docs.docker.com/engine/installation/linux/centos/
# online document

sudo yum update
# Make sure your existing packages are up-to-date.

curl -fsSL https://get.docker.com/ | sh
# Run the Docker installation script.

sudo systemctl enable docker.service
# Enable the service.

sudo systemctl start docker
# Start the Docker daemon.

sudo docker run hello-world
# Verify docker is installed correctly by running a test image in a container.
```

### 安装docker-compose

```
https://docs.docker.com/compose/install/
# online document

curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# The following is an example command illustrating the format

chmod +x /usr/local/bin/docker-compose
# Apply executable permissions to the binary

docker-compose --version
# Test the installation.
```

### 安装harbor

```
https://github.com/vmware/harbor/blob/master/docs/installation_guide.md
# online document

tar xvf harbor-online-installer-<version>.tgz
install.sh
```
