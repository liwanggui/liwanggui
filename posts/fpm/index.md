# RPM 快速打包


回顾下安装软件的三种方式：

1. 编译安装软件，优点是可以定制化安装目录、按需开启功能等，缺点是需要查找并实验出适合的编译参数，诸如 MySQL 之类的软件编译耗时过长。
2. yum 安装软件，优点是全自动化安装，不需要为依赖问题发愁了，缺点是自主性太差，软件的功能、存放位置都已经固定好了，不易变更。
3. 编译源码，根据自己的需求做成定制 RPM 包 --> 搭建内网yum仓库 --> yum安装。结合前两者的优点，暂未发现什么缺点。可能的缺点就是 RPM 包的通用性差，只能适用于本公司的环境。另外一般人不会定制 RPM 包。这是中大型互联网企业运维自动化的必要技能。

这里也不介绍 rpm 的概念，想了解的朋友可以查看 [http://www.ibm.com/developerworks/cn/linux/l-rpm/](http://www.ibm.com/developerworks/cn/linux/l-rpm/)

这里也不介绍 `rpmbuild` 这个打包工具了，想了解的朋友自行谷歌百度。但我不建议大家花太多的时间去学习这个命令，比较晦涩，而且我会在下面介绍更简单的命令。

## FPM打包工具

FPM 的作者是 `jordansissel` [FPM 的 github](https://github.com/jordansissel/fpm)

FPM 功能简单说就是将一种类型的包转换成另一种类型。

### 1. 支持的源类型包

1. dir 将目录打包成所需要的类型，可以用于源码编译安装的软件包
2. rpm 对rpm进行转换
3. gem 对rubygem包进行转换
4. python 将python模块打包成相应的类型

### 2. 支持的目标类型包

- rpm 转换为rpm包
- deb 转换为deb包
- solaris 转换为solaris包
- puppet 转换为puppet模块

### 3. FPM 安装

fpm 是 ruby 写的，因此系统环境需要 ruby，且 ruby 版本号大于1.8.5。rpm 包太旧，安装后会提示版本问题。所以采用编译安装方式：

```bash
# 下载 ruby 源码包
wget -c https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.4.tar.gz
tar xzf ruby-2.3.4.tar.gz
cd ruby-2.3.4
./configure && make && make install

# 添加阿里云的Rubygems仓库，外国的源慢
gem sources -a http://mirrors.aliyun.com/rubygems/
# 移除原生的Ruby仓库
gem sources --remove http://rubygems.org/
# 安装fpm
gem install fpm
```
### 4. FPM参数

详细使用见 `fpm --help` 常用参数

```
-s 指定源类型
-t 指定目标类型，即想要制作为什么包
-n 指定包的名字
-v 指定包的版本号
-C 指定打包的相对路径 Change directory to here before searching forfiles
-d 指定依赖于哪些包
-f 第二次打包时目录下如果有同名安装包存在，则覆盖它
-p 输出的安装包的目录，不想放在当前目录下就需要指定
--post-install 软件包安装完成之后所要运行的脚本；同--after-install
--pre-install 软件包安装完成之前所要运行的脚本；同--before-install
--post-uninstall 软件包卸载完成之后所要运行的脚本；同--after-remove
--pre-uninstall 软件包卸载完成之前所要运行的脚本；同--before-remove
```

## 使用实例--实战定制nginx的RPM包

### 1. 安装nginx

```bash
yum -y install pcre-devel openssl-devel
useradd -M -s /sbin/nologin nginx
tar xf nginx-1.6.2.tar.gz
cd nginx-1.6.2
./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_stub_status_module
make && make install
```

### 2. 编写脚本

```bash
[root@oldboy ~]# cd /server/scripts/
[root@oldboy scripts]# vim nginx_rpm.sh # 这是安装完rpm包要执行的脚本
#!/bin/bash
useradd -M -s /sbin/nologin nginx
```

### 3. 打包

```bash
[root@oldboy ~]# fpm -s dir -t rpm -n nginx -v 1.6.2 \
-d 'pcre-devel,openssl-devel' \
--post-install /server/scripts/nginx_rpm.sh \
-f /usr/local/nginx
no value for epoch is set, defaulting to nil {:level=>:warn}
no value for epoch is set, defaulting to nil {:level=>:warn}
Created package {:path=>"nginx-1.6.2-1.x86_64.rpm"}
[root@oldboy ~]# ll -h nginx-1.6.2-1.x86_64.rpm
-rw-r--r-- 1 root root 6.7M Nov 1 10:02 nginx-1.6.2-1.x86_64.rpm
```

### 4. 安装rpm包

**安装rpm包的三种方法：**

*①rpm命令安装*

```bash
[root@LB-nginx-01 ~]# rpm -ivh nginx-1.6.2-1.x86_64.rpm
error: Failed dependencies:
pcre-devel is needed by nginx-1.6.2-1.x86_64
openssl-devel is needed by nginx-1.6.2-1.x86_64
```

> 但会报如上依赖错误，需要先yum安装依赖才能安装 rpm 包

*yum 命令安装rpm包*

```bash
yum -y localinstall nginx-1.6.2-1.x86_64.rpm
```

> 这个命令会自动先安装 rpm 包的依赖，然后再安装 rpm 包。可以配合自建的 yum 本地仓库使用

## 注意事项

*相对路径问题*

```# 相对路径
[root@oldboy nginx]# fpm -s dir -t rpm -n nginx -v 1.6.2 .
no value for epoch is set, defaulting to nil {:level=>:warn}
no value for epoch is set, defaulting to nil {:level=>:warn}
Created package {:path=>"nginx-1.6.2-1.x86_64.rpm"}
[root@oldboy nginx]# rpm -qpl nginx-1.6.2-1.x86_64.rpm
/client_body_temp
/conf/extra/dynamic_pools
/conf/extra/static_pools
…………

# 绝对路径
[root@oldboy ~]# fpm -s dir -t rpm -n nginx -v 1.6.2 /usr/local/nginx
no value for epoch is set, defaulting to nil {:level=>:warn}
no value for epoch is set, defaulting to nil {:level=>:warn}
Created package {:path=>"nginx-1.6.2-1.x86_64.rpm"}
[root@oldboy ~]# rpm -qpl nginx-1.6.2-1.x86_64.rpm
/usr/local/nginx/client_body_temp
/usr/local/nginx/conf/extra/dynamic_pools
/usr/local/nginx/conf/extra/static_pools
/usr/local/nginx/conf/fastcgi.conf
/usr/local/nginx/conf/fastcgi.conf.default
…………
```

> 注：fpm 类似 tar 打包一样，只是 fpm 打的包能够被 yum 命令识别而已。
