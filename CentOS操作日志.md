# 更新CentOS Yum镜像源和软件包

##### 备份系统自带yum源配置文件/etc/yum.repos.d/CentOS-Base.repo
```bash 	
[root@localhost ~]# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```
##### 进入镜像源文件夹
```bash 
[root@localhost ~]# cd /etc/yum.repos.d/
```
##### 下载相应的镜像源文件，如163，aliyun
```bash 
# 网易云
[root@localhost yum.repos.d]# wget -O CentOS-Base.repo http://mirrors.163.com/.help/CentOS版本号-Base-163.repo
# 阿里云
[root@localhost yum.repos.d]# wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-版本号.repo

# wget -O参数为更名
```
##### 生成新镜像源缓存
```bash 
[root@localhost yum.repos.d]# yum makecache
```
##### 更新软件包
```bash 
[root@localhost yum.repos.d]# yum -y update
```


# 更新CentOS 系统内核(ELRepo)

##### 导入公钥
```bash
[root@localhost ~]# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org 
```

##### 安装ELRepo 可以根据 ELRepo 的 Index 页面查看最新rpm文件


[ELRepo-CentOS7](https://elrepo.org/linux/kernel/el7/x86_64/RPMS/) 
\
https://elrepo.org/linux/kernel/el版本号/x86_64/RPMS/
```bash 
[root@localhost ~]# rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
```

##### 安装新版本内核
```bash
[root@localhost ~]# yum --enablerepo=elrepo-kernel install kernel-ml -y
```

##### 备份系统Grub配置文件
```bash
[root@localhost ~]# cp /etc/default/grub /etc/default/grub.bak
```

##### 查询当前已安装的内核
```bash
[root@localhost ~]# rpm -qa | grep kernel
or
[root@localhost ~]# grub2-editenv list
```

##### 查询当前的内核启动顺序
```bash
[root@localhost ~]# sudo egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
CentOS Linux (4.4.196-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-1062.4.1.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-1062.1.1.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-1062.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-2987df5e8bede334c417bfb01b7b5725) 7 (Core)
CentOS Linux (0-rescue-c2bf71aa0bb54391adb6375abeaa5e4f) 7 (Core)
```

##### 修改Grub2启动引导管理，更改新内核优先启动
```bash
[root@localhost ~]# sudo grub2-set-default 0
or
[root@localhost ~]# grub2-set-default "CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)" ;
# 数字0根据查询出来的启动顺序从0开始算起
```

##### 修改Grub启动管理，更改新内核优先启动 (不知原因，个人使用时无效)
```bash
# [root@localhost ~]# vim /etc/default/grub

# GRUB_TIMEOUT=5
# GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
# GRUB_DEFAULT=saved      #把这里的saved改成0
# GRUB_DISABLE_SUBMENU=true
# GRUB_TERMINAL_OUTPUT="console"
# GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet net.ifnames=0"
# GRUB_DISABLE_RECOVERY="true"

```
##### 运行grub2-mkconfig命令来重新创建内核配置 (不知原因，个人使用时无效)

```bash
# [root@localhost ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
# Generating grub configuration file ...
# Found linux image: /boot/
# ......
# done

```


##### 安装内核头和devel
```bash
uname -a ; rpm -qa kernel\* | sort
sudo yum install "kernel-devel-uname-r == $(uname -r)"
# [root@localhost ~]# yum install kernel-headers
#查询内核头
yum list | grep kernel-headers
# 安装内核头
dkms status
dpkg-query -s linux-headers-$(uname -r) 
sudo yum install linux-headers-$(uname -r) 
```


##### 重启服务器
```bash
[root@localhost ~]# reboot
```

##### 查询系统版本
```bash
[root@localhost ~]# cat /etc/redhat-release
```

##### 查询系统内核版本
```bash
[root@localhost ~]# uname -sr
```
#### PS 内核版本说明
```bash
# ELRepo有两种类型的Linux内核包，kernel-lt和kernel-ml。 他们之间有什么区别？
# kernel-ml软件包是根据Linux Kernel Archives的主线稳定分支提供的源构建的。 内核配置基于默认的RHEL-7配置，并根据需要启用了添加的功能。 这些软件包有意命名为kernel-ml，以免与RHEL-7内核发生冲突，因此，它们可以与常规内核一起安装和更新。
# kernel-lt包是从Linux Kernel Archives提供的源代码构建的，就像kernel-ml软件包一样。 不同之处在于kernel-lt基于长期支持分支，而kernel-ml基于主线稳定分支。
```


# 修改系统时区

```bash
# 方法1
[root@localhost ~]# cp  /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime

# 方法2：    
# 列出时区
[root@localhost ~]# timedatectl list-timezones
# 选择时区
[root@localhost ~]# timedatectl set-timezone Asia/Shanghai

# 方法3： 
[root@localhost ~]# tzselect
```

# 安装man中文手册
```bash
# 查询中文手册安装包
[root@localhost ~]# yum list |grep man.*zh
# 安装中文手册安装包
[root@localhost ~]# sudo yum install man-pages-zh-CN.noarch
# 添加中文手册使用别名cman
[root@localhost ~]# vi .bashrc
[root@localhost ~]# alias cman='man -M /usr//share/man/zh_CN'
[root@localhost ~]# source .bashrc
# 测试
[root@localhost ~]# cman ls
```


# 开启BBR加速

##### 更改sysctl.conf文件
```bash
# 老版本Linux可以使用，新版本sysctl.conf文件需要到/etc/sysctl.d文件夹下新建conf文件去进行配置

# 老版本
[root@localhost ~]# vim /etc/sysctl.conf 
# 在文件末尾添加如下内容
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

# 或者执行
[root@localhost ~]# echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
[root@localhost ~]# echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
[root@localhost ~]# sudo sysctl -p


# 新版本
[root@localhost ~]# cd /etc/sysctl.d
[root@localhost ~]# cp 99-sysctl.conf sysctl.conf

# 或者执行
[root@localhost ~]# echo 'net.core.default_qdisc=fq' | sudo tee -a sysctl.conf
[root@localhost ~]# echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a sysctl.conf
[root@localhost ~]# sudo sysctl -p
```

##### 添加系统参数
```bash
[root@localhost ~]# sysctl -p
```

##### 检查是否开启成功
```bash 
[root@localhost ~]# lsmod | grep bbr
tcp_bbr                20480  2

[root@localhost ~]# sysctl net.ipv4.tcp_available_congestion_control
net.ipv4.tcp_available_congestion_control = reno cubic bbr
```

# 安装Docker
##### 准备工作
```bash
# 检查系统内核，需高于3.1 
[root@localhost ~]# uname -sr
# 更新系统yum为最新的软件包
[root@localhost ~]# sudo yum -y update
# 如果有旧版本可以用该命令进行卸载
[root@localhost ~]# sudo yum remove docker docker-common docker-selinux docker-engine
·······························································
# 强制删除
[root@localhost ~]# yum remove docker docker-common docker-selinux docker-engine -y
[root@localhost ~]# /etc/systemd -name '*docker*' -exec rm -f {} ;
[root@localhost ~]# find /etc/systemd -name '*docker*' -exec rm -f {} \;
[root@localhost ~]# find /lib/systemd -name '*docker*' -exec rm -f {} \;
·······························································
# 安装相关依赖
[root@localhost ~]# sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# 设置docker 的yum源(国内主机的话可以用阿里云的(docker-yum源)[http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo])
[root@localhost ~]# sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
##### 安装Docker
```bash 
# 查看docker版本
[root@localhost ~]# yum list docker-ce --showduplicates | sort -r
 * updates: mirrors.aliyun.com
Loaded plugins: fastestmirror, langpacks
 * extras: mirrors.aliyun.com
 * epel: d2lzkl7pfhq30w.cloudfront.net
 * elrepo: ftp.ne.jp
docker-ce.x86_64            3:19.03.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.3-3.el7                    
......
Determining fastest mirrors
 * base: mirrors.aliyun.com
Available Packages
```
```bash
# 设置稳定的仓库。
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 安装最新稳定版本
$ sudo yum install docker-ce docker-ce-cli containerd.io
[root@localhost ~]# yum -y install docker-ce
```
##### 设置开机启动
```bash
[root@localhost ~]# sudo systemctl start docker
[root@localhost ~]# sudo systemctl enable docker
```
##### 查询版本和验证安装Docker Engine-Community 正确与否
```bash
[root@localhost ~]# sudo docker version
[root@localhost ~]# sudo docker run hello-world
```

##### 安装Docker-compose [官网Compose安装说明](https://docs.docker.com/compose/install/)
```bash
# 通过查询官网 Install Compose on Linux systems找到最新版本安装命令
[root@localhost ~]# sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
[root@localhost ~]# sudo chmod +x /usr/local/bin/docker-compose
[root@localhost ~]# sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
# 检查版本
[root@localhost ~]# docker-compose --version
```
##### DockerFile的创建使用和打包
```bash
# 以node项目为例
# 获取一个node项目，进入到该项目中
# 编辑Dockerfile文件 
[root@localhost ~]# vi Dockerfile
# 工作镜像基础包
FROM node:13

# 创建镜像的app项目目录
WORKDIR /usr/src/app

# 复制项目到镜像的项目目录中
COPY ./ /usr/src/app
# 执行相关安装node依赖语句或者其他项目的初始化语句等操作
RUN npm i
# 默认暴露端口
EXPOSE 2333
# 启动命令
CMD [ "npm", "start" ]

# 镜像打包
[root@localhost ~]# sudo docker build -t project_name .
# 编辑compose启动文件
[root@localhost ~]# vi docker-compose.yml
version: "3"
services:
  cah_client:
    image: cah_client
    container_name: cah_client
    restart: always
    ports:
      - "3000:3000"
  cah_server:
    image: cah_server
    container_name: cah_server
    restart: always
    ports:
      - "2333:2333"

# 镜像运行 编辑docker-compose.yml文件
[root@localhost ~]# sudo docker-compose up -d

# mysql 显示乱码
[root@localhost ~]# mysql> Mysql SET NAMES 'utf8';
# 它相当于下面的三句指令：
[root@localhost ~]# mysql> SET character_set_client = utf8;
[root@localhost ~]# mysql> SET character_set_results = utf8;
[root@localhost ~]# mysql> SET character_set_connection = utf8;

```
docker run --name php-fpm -v /home/opc/Service/nginx/html:/var/www/html -p 9000:9000 -d php:7.3-fpm

# 安装Git
```bash
[root@localhost ~]# sudo yum -y install git
# 配置基本信息
[root@localhost ~]# git config --global user.name "Bitamin.kim"
[root@localhost ~]# git config --global user.email Bitamin.kim@gmail.com
//查看配置
[root@localhost ~]# git config --list
user.name=Bitamin.kim
user.email=Bitamin.kim@gmail.com
# 配置SSH连接公私钥
#[root@localhost ~]# ssh-keygen -t rsa -b 4096 -C <your_email@example.com>
[root@localhost ~]# ssh-keygen -t rsa -b 4096 -C Bitamin.kim@gmail.com
# 如果是个人已有的密钥需要对其更改权限为700
[root@localhost ~]# chmod 700 id_rsa
# 如果出现 Could not open a connection to your authentication agent.
# 使用该命令开启公钥管理
[root@localhost ~]# ssh-agent bash
# 添加公钥
[root@localhost ~]# ssh-add ~/.ssh/id_rsa
# 测试连接
[root@localhost ~]# ssh -T git@github.com
Warning: Permanently added the RSA host key for IP address '52.192.72.89' to the list of known hosts.
Hi BitaminKim! You've successfully authenticated, but GitHub does not provide shell access.
```

# 安装node
```bash
# 方法一
# 安装编译用的依赖 可能需要升级 该方法不推荐
[root@localhost ~]# sudo yum install gcc gcc-c++
# 官网获取最新的tar.gz源码压缩包下载地址 https://nodejs.org/en/download/   Source Code
[root@localhost ~]# wget https://nodejs.org/dist/v版本号/node-v版本号.tar.gz
# 解压
[root@localhost ~]# tar zxvf node-v版本号.tar.gz
# 进入解压后的目录执行配置
[root@localhost ~]# cd node-v版本号
[root@localhost ~]# ./configure

# 进行编译


# 方法二 EPEL
# http://dl.fedoraproject.org/pub/epel/7/，下载rpm文件
[root@localhost ~]# sudo rpm -ivh https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-12.noarch.rpm
# 查询node最新版本地址https://github.com/nodesource/distributions
[root@localhost ~]# sudo curl -sL https://rpm.nodesource.com/setup_13.x | sudo bash -
[root@localhost ~]# sudo yum install -y nodejs
# 检查安装
[root@localhost ~]# node -v
[root@localhost ~]# npm -v

```

#安装Wireguard
```bash
[root@localhost ~]# modprobe wireguard && lsmod | grep wireguard
```
