# NGINX 安装


本文将介绍 NGINX 常用的二种安装方式，分为二进制包和源码编译，根据自己的使用的场景选择即可。

## 二进制包

二进制方式安装比效简单，直接使用系统包管理工具安装即可

*Ubuntu*

```bash
apt install nginx
```

*CentOS*

```bash
yum install nginx
```

## 源码编译

### 准备编译环境

本文使用 Ubuntu 20.04.2 LTS 编译安装.

```bash
root@ubuntu:~# apt install gcc make openssl libssl-dev libpcre3-dev zlib1g-dev
root@ubuntu:~# cd /usr/local/src
root@ubuntu:/usr/local/src# wget http://nginx.org/download/nginx-1.18.0.tar.gz
root@ubuntu:/usr/local/src# tar xzf nginx-1.18.0.tar.gz
```

### 开始编译安装

```bash
root@ubuntu:/usr/local/src# cd nginx-1.18.0
root@ubuntu:/usr/local/src/nginx-1.18.0# useradd -r www
root@ubuntu:/usr/local/src/nginx-1.18.0# ./configure --prefix=/usr/local/nginx \
--user=www \
--group=www \
--with-threads \
--with-file-aio \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_addition_module \
--with-http_sub_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_auth_request_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_degradation_module \
--with-http_slice_module \
--with-http_stub_status_module \
--with-stream \
--with-stream_ssl_module \
--with-stream_realip_module \
--with-stream_ssl_preread_module
root@ubuntu:/usr/local/src/nginx-1.18.0# make && make install
```

### Nginx 服务管理

*启动 nginx 服务*

```bash
root@ubuntu:/usr/local/nginx# /usr/local/nginx/sbin/nginx
root@ubuntu:/usr/local/nginx# curl -I localhost
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Thu, 01 Apr 2021 08:16:00 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Thu, 01 Apr 2021 08:07:38 GMT
Connection: keep-alive
ETag: "60657f4a-264"
Accept-Ranges: bytes
```

*管理 nginx 服务*

```bash
# 测试 nginx 配置文件语法
root@ubuntu:~# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
# 重载 nginx 配置
root@ubuntu:~# /usr/local/nginx/sbin/nginx -s reload
# 停止 nginx 服务
root@ubuntu:~# /usr/local/nginx/sbin/nginx -s stop
```

### Systemd 配置

```bash
root@ubuntu:~# cat > /usr/lib/systemd/system/nginx.service <<EOF
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```
