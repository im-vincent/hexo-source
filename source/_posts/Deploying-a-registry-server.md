title: Deploying a registry server
date: 2016-03-31 15:12:50
tags: [docker]
---

1.    制作自签署证书

如果你有知名CA签署的证书，那么这步可直接忽略。

```
$ openssl req -newkey rsa:2048 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
Generating a 2048 bit RSA private key
..............+++
............................................+++
writing new private key to 'certs/domain.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Liaoning
Locality Name (eg, city) []:shenyang
Organization Name (eg, company) [Internet Widgits Pty Ltd]:foo
Organizational Unit Name (eg, section) []:bar
Common Name (e.g. server FQDN or YOUR name) []:mydockerhub.com
Email Address []:bigwhite.cn@gmail.com

```

2.    启动Secure Registry
启动带证书的Registry：

```
$ docker run -d -p 5000:5000 --restart=always --name registry \
  -v `pwd`/data:/var/lib/registry \
  -v `pwd`/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```

3.    docker client安装CA证书

```
$ sudo mkdir -p /etc/docker/certs.d/mydockerhub.com:5000
$ sudo cp certs/domain.crt /etc/docker/certs.d/mydockerhub.com:5000/ca.crt
$ sudo service docker restart //安装证书后，重启Docker Daemon
```

4.    使用docker-compose启动
```
[root@mydockerhub Compose]# cat registry.yaml
registry:
  restart: always
  image: registry:2
  ports:
    - 5000:5000
  environment:
    REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
    REGISTRY_HTTP_TLS_KEY: /certs/domain.key
#    REGISTRY_AUTH: htpasswd
#    REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
#    REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
  volumes:
    - /home/wangxi/data:/var/lib/registry
    - /home/wangxi/certs:/certs
#    - /path/auth:/auth                                    

[root@mydockerhub Compose]# docker-compose -f registry.yaml up -d 
```
