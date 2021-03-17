# 安装 Redis


## Redis 使用场景介绍

Memcached：多核的缓存服务，更加适合于多用户并发访问次数较少的应用场景

Redis：单核的缓存服务，单节点情况下，更加适合于少量用户，多次访问的应用场景。 redis 一般是单机多实例架构，配合 redis 集群出现。

## 安装 Reis

```bash
cd /usr/local/src
wget http://download.redis.io/releases/redis-3.2.12.tar.gz
tar xzf redis-3.2.12.tar.gz
yum -y install gcc automake autoconf libtool make
cd /usr/local/src/redis-3.2.12
make
make install
```

## 配置 Redis

### 安装 Redis 服务

使用源码包中的 `install_server.sh` 脚本工具，安装 redis 服务

```bash
[root@10-7-171-239 utils]# bash install_server.sh
Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379]
Selecting default: 6379
Please select the redis config file name [/etc/redis/6379.conf]
Selected default - /etc/redis/6379.conf
Please select the redis log file name [/var/log/redis_6379.log]
Selected default - /var/log/redis_6379.log
Please select the data directory for this instance [/var/lib/redis/6379]
Selected default - /var/lib/redis/6379
Please select the redis executable path [/usr/local/bin/redis-server]
Selected config:
Port           : 6379
Config file    : /etc/redis/6379.conf
Log file       : /var/log/redis_6379.log
Data dir       : /var/lib/redis/6379
Executable     : /usr/local/bin/redis-server
Cli Executable : /usr/local/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6379.conf => /etc/init.d/redis_6379
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!
```

### 连接测试 Redis

```bash
$ systemctl start redis_6379
$ systemctl status redis_6379.service
● redis_6379.service - LSB: start and stop redis_6379
   Loaded: loaded (/etc/rc.d/init.d/redis_6379; bad; vendor preset: disabled)
   Active: active (exited) since 三 2021-02-24 12:05:58 HKT; 4s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 24570 ExecStart=/etc/rc.d/init.d/redis_6379 start (code=exited, status=0/SUCCESS)

2月 24 12:05:58 10-7-171-239 systemd[1]: Starting LSB: start and stop redis_6379...
2月 24 12:05:58 10-7-171-239 redis_6379[24570]: /var/run/redis_6379.pid exists, process is already running or crashed
2月 24 12:05:58 10-7-171-239 systemd[1]: Started LSB: start and stop redis_6379.
$ redis-cli
127.0.0.1:6379> ping
PONG
```

### 调整 redis 配置

```bash
$ egrep -v '^#|^$' /etc/redis/6379.conf
# 绑定的 ip 地址， 默认只绑定 127.0.0.1 不允许外部客户端连接
bind 127.0.0.1 10.7.171.239 
protected-mode yes
port 6379   
tcp-backlog 511
timeout 0
tcp-keepalive 300
# 是否后台运行
daemonize yes
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile /var/log/redis_6379.log
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
# RDB 持久化数据文件, 存储在 dir 选项配置的目录下
dbfilename dump.rdb
# 数据持久化存储路径
dir /var/lib/redis/6379
# 连接时验证的密码
requirepass S4Ea0lFLwJjehB91
# 最大使用内存大小
maxmemory 512m

slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
```

