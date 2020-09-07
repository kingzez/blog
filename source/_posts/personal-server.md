---
title: 个人 Server 环境搭建(CentOS)
date: 2020-09-04 16:37:48
tags: linux
---
> 全面 docker 化…em 尽量
> 持续更新…

### yum

```bash
# 更新系统
sudo yum -y update

sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
### nodejs

```bash
# 使用 NodeSource 二进制发行版 安装nodejs12
curl -sL https://rpm.nodesource.com/setup_12.x | sudo bash -

sudo yum clean all && sudo yum makecache fast
sudo yum install -y gcc-c++ make
sudo yum install -y nodejs
```
### docker

```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io

systemctl enable --now docker
```
### mongodb

```bash
docker pull mongo:3.5

mkdir -p /opt/data/mongo

docker run -d -p 27017:27017 -v /opt/data/mongo:/data/db --restart always --name mongo mongo:3.
```
### nginx

```bash
sudo yum install epel-release
sudo yum install nginx

# 使用系统启动
sudo systemctl start nginx

# 使用docker
docker pull nginx
docker run -d --network host --name nginx -v /etc/nginx/:/etc/nginx nginx

# 修改后检查
docker exec nginx nginx -t

# 重启nginx
docker restart nginx

# 查看日志
docker logs -f --tail 10 nginx
```

### https
> [https://certbot.eff.org/lets-encrypt/centosrhel7-nginx](https://certbot.eff.org/lets-encrypt/centosrhel7-nginx)

```bash
# 执行前提 关闭nginx服务
sudo certbot certonly --nginx -d kingzez.com
```

 
```nginx
# 备忘
ssl_certificate			/etc/nginx/kingzez.com.pem;
ssl_certificate_key /etc/nginx/kingzez.com.key.pem;
ssl_trusted_certificate /etc/nginx/kingzez.com.pem;
ssl_ciphers         EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH;
ssl_protocols       TLSv1.1 TLSv1.2;

server {
  server_name api.kingzez.com;
  listen 443 ssl;

  location / {
    proxy_pass http://localhost:3000/;
  }

  location /test/ {
    proxy_pass http://localhost:4000/;
  }
}
```
