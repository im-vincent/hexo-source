title: CentOS6.4不删除yum下载后的rpm包
date: 2015-12-17 16:17:53
tags: [linux]
---

yum 默认情况下，升级或者安装后，会删除下载的rpm包。
可以设置升级后不删除下载的rpm包

```
vi /etc/yum.conf
[main]
cachedir=/var/cache/yum
keepcache=0
将 keepcache=0 修改为 keepcache=1， 安装或者升级后，
在目录 /var/cache/yum/i386/6/base/packages 下就会有下载的 rpm 包。
```
