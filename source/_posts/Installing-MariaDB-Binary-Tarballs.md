title: Installing MariaDB Binary Tarballs
date: 2015-11-05 14:18:48
tags: [mysql]
---

[Installing MariaDB Binary Tarballs](https://mariadb.com/kb/en/mariadb/installing-mariadb-binary-tarballs/ "Title")

Installing MariaDB as root in /usr/local/mysql
If you have root access to the system, you probably want to install MariaDB under the user and group 'mysql' (to keep compatibility with MySQL installations):

```
groupadd mysql
useradd -g mysql mysql
cd /usr/local
tar -zxvpf /path-to/mariadb-VERSION-OS.tar.gz
ln -s mariadb-VERSION-OS mysql
cd mysql
./scripts/mysql_install_db --user=mysql
chown -R root .
chown -R mysql data

```

```
./bin/mysqld_safe --user=mysql &
or
./bin/mysqld_safe --defaults-file=~/.my.cnf --user=mysql &
```

```
export PATH=$PATH:/usr/local/mysql/bin/
```

```
cp support-files/mysql.server /etc/init.d/mysql.server
cp ./support-files/my-medium.cnf /etc/my.cnf
```
