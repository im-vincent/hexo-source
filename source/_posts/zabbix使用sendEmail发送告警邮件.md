title: zabbix使用sendEmail发送告警邮件
date: 2015-12-14 15:22:47
tags: [linux,zabbix]
---
zabbix使用sendEmail发送告警邮件



* sendEmail是一个免费、轻量级、命令行的SMTP邮件客户端。
* 如果你需要使用命令行方式发送邮件，那么sendEmail是非常完美的选择：使用简单、功能强大。

```
cd /usr/local/src/
wget http://caspian.dotconf.net/menu/Software/SendEmail/sendEmail-v1.56.tar.gz
tar xvfz sendEmail-v1.56.tar.gz
cp sendEmail-v1.56/sendEmail /usr/local/bin/
chmod +x /usr/local/bin/sendEmail 

```

* zabbix服务器端发送邮件脚本

```
mkdir -p /etc/zabbix/alertscripts
chown -R zabbix:zabbix /etc/zabbix/alertscripts
echo "AlertScriptsPath=/etc/zabbix/alertscripts" >> /etc/zabbix/zabbix_server.conf
重启服务
/etc/init.d/zabbix-server stop
/etc/init.d/zabbix-server start
```

* 创建发送邮件脚本

```
vim /etc/zabbix/alertscripts/SendEmail.sh

#!/bin/bash
SMTP_server='smtp.163.com'    # SMTP服务器
username='zabbix@163.com'     # 用户名
password='zabbix'             # 密码
from_email_address='zabbix@163.com' # 发件人Email地址
to_email_address="$1"               # 收件人Email地址，zabbix传入的第一个参数
message_subject_utf8="$2"           # 邮件标题，zabbix传入的第二个参数
message_body_utf8="$3"              # 邮件内容，zabbix传入的第三个参数

# 转换邮件标题为GB2312，解决邮件标题含有中文，收到邮件显示乱码的问题。
message_subject_gb2312=`iconv -t GB2312 -f UTF-8 << EOF
$message_subject_utf8
EOF`
[ $? -eq 0 ] && message_subject="$message_subject_gb2312" || message_subject="$message_subject_utf8"

# 转换邮件内容为GB2312
message_body_gb2312=`iconv -t GB2312 -f UTF-8 << EOF
$message_body_utf8
EOF`
[ $? -eq 0 ] && message_body="$message_body_gb2312" || message_body="$message_body_utf8"

# 发送邮件
sendEmail='/usr/local/bin/sendEmail'
$sendEmail -s "$SMTP_server" -xu "$username" -xp "$password" -f "$from_email_address" -t "$to_email_address" -u "$message_subject" -m "$message_body" -o message-content-type=text -o message-charset=gb2312

```
* 可以先简单测试下

```
chown zabbix:zabbix /etc/zabbix/alertscripts/SendEmail.sh
chmod +x /etc/zabbix/alertscripts/SendEmail.sh 

[root@se120 alertscripts]# ./SendEmail.sh wangxi@cis.com.cn sdfs sdfsdf
Dec 14 11:13:24 se120 sendEmail[80340]: Email was sent successfully!
```

* 配置Email告警方式
* Administration -> Media types 
![](http://i4.tietuku.com/45f09cddc6a664fbs.jpg)




