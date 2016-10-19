title: Kubernetes Cluster with 3 minions on CentOS 7
date: 2016-02-23 15:19:42
tags: [Kubernetes]
---

### Prerequisites

| 名称         | IP地址           | 组件                                       |
| :--------- | -------------- | ---------------------------------------- |
| k8s-master | 172.16.153.101 | etcd、 kube-apiserver、kube-controller- manager、 kube-scheduler |
| k8s-node1  | 172.16.153.111 | docker、 kubelet、 kube-proxy              |
| k8s-node2  | 172.16.153.112 | docker、 kubelet、 kube-proxy              |

```
hostnamectl set-hostname k8s-master
#设置主机名

sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
#关闭selinux

systemctl stop firewalld
systemctl disable firewalld
#关闭防火墙

yum -y install ntp
systemctl start ntpd
systemctl enable ntpd
#ntp时间同步
```


### Setting up the Kubernetes Master
#### 安装etcd、kubernetes

    ```
     yum -y install etcd kubernetes-master
    ```

#### 配置etcd

    ```
     [root@k8s-master ~]# rpm -qc etcd
     /etc/etcd/etcd.conf

     [root@k8s-master ~]# grep -v ^# /etc/etcd/etcd.conf
     ETCD_NAME=default
     ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
     ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
     ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"
    ```

     ​
#### 配置apiserver

    ```
     [root@k8s-master ~]# rpm -qa|grep kube
     kubernetes-master-1.2.0-0.11.git738b760.el7.x86_64
     kubernetes-1.2.0-0.11.git738b760.el7.x86_64

     [root@k8s-master ~]# grep -v "^#" /etc/kubernetes/config
     KUBE_LOGTOSTDERR="--logtostderr=true"
     KUBE_LOG_LEVEL="--v=0"
     KUBE_ALLOW_PRIV="--allow-privileged=false"
     KUBE_MASTER="--master=http://172.16.153.101:8080"

     [root@k8s-master ~]# grep -v "^#" /etc/kubernetes/apiserver
     KUBE_API_ADDRESS="--address=0.0.0.0"
     KUBE_ETCD_SERVERS="--etcd-servers=http://172.16.153.101:2379"
     KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
     KUBE_ADMISSION_CONTROL="--admission-  control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
     KUBE_API_ARGS=""
     #注意，--admission-control参数里没有ServiceAccount
    ```

     ​

#### 启动服务

    ```
     for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
         systemctl restart $SERVICES
         systemctl enable $SERVICES
         systemctl status $SERVICES
     done
    ```

     ​

#### 配置flannel网络信息在etcd

     ```
     etcdctl mk /atomic.io/network/config '{"Network":"172.17.0.0/16"}'
     #/atomic.io/network/config这个信息要和后面flannel配置信息一致，如果配置错误删除etcd不能删除这个信息。
     ```

     ​
### Setting up Kubernetes Minions (Nodes)

#### 安装flannel、Kubernetes

    ```
     yum -y install flannel kubernetes-node
    ```

#### 配置flannel

    ```
     [root@localhost ~]# grep -v ^# /etc/sysconfig/flanneld
     FLANNEL_ETCD="http://172.16.153.101:2379"
     FLANNEL_ETCD_KEY="/atomic.io/network"
    ```

#### 配置kubernetes

    ```
     [root@localhost ~]# grep -v ^# /etc/kubernetes/config
     KUBE_LOGTOSTDERR="--logtostderr=true"
     KUBE_LOG_LEVEL="--v=0"
     KUBE_ALLOW_PRIV="--allow-privileged=false"
     KUBE_MASTER="--master=http://172.16.153.101:8080"

     root@localhost ~]# grep -v ^# /etc/kubernetes/kubelet
     KUBELET_ADDRESS="--address=0.0.0.0"
     KUBELET_HOSTNAME="--hostname-override=172.16.153.111"
     KUBELET_API_SERVER="--api-servers=http://172.16.153.101:8080"
     KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"
     KUBELET_ARGS="--pod-infra-container-image=index.alauda.cn/googlecontainer/pause"
     #这里自定义设置了一个pause，防止被墙下载不过来
    ```

#### 启动node端服务（node1、node2）

    ```
     for SERVICES in kube-proxy kubelet docker flanneld; do
         systemctl restart $SERVICES
         systemctl enable $SERVICES
         systemctl status $SERVICES
     done
    ```

     ​

     ​
