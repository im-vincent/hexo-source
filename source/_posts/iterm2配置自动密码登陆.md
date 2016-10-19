title: iterm2配置自动密码登陆
date: 2016-10-18 16:31:21
tags: [linux]
---
### 下载安装sshpass

```
cd /usr/local/src
curl -OL http://jaist.dl.sourceforge.net/project/sshpass/sshpass/1.06/sshpass-1.06.tar.gz
tar xvfz sshpass-1.06.tar.gz
cd sshpass-1.06
./configure && make && make install
```

### 配置iterm2

profiles => open profiles => edit profile

![](http://ww3.sinaimg.cn/large/6e46250bjw1f8wi5tzaqij20fk0cb0tw.jpg)

新建一个连接，在command里设置。因为iterm2默认环境变量是从/usr/bin和/usr/sbin里取的。所以我们要加绝对路径

```
/usr/local/bin/sshpass -p password ssh -o StrictHostKeyChecking=no user@ip
```
