---
title: linux-软件包安装与管理
date: 27-12-2020
categories: 
- linux
tags:
- note
---

## 一.软件包管理器
#### 1.概念
 - centos、redhat使用yum包管理器，软件安装包格式为rpm
 - debian、ubuntu使用apt包管理器，软件安装包格式为deb

#### 2.rpm包格式
 apr-util-1.5.2-6.el7.x86_64.rpm  
 软件名称 软件版本 系统版本 平台
#### 3.linux中的设备文件：/dev
 ```
cd /dev //进入设备目录
mount /dev/sr0 /mnt/ //将设备/dev/sr0挂载到一个空白的目录
cd /mnt
cd Packages //所有rpm安装包在/mnt/Packages目录下
```
#### 4.什么是挂载？
 在windows上，把U盘插到电脑上，windows上出现u盘的盘符，就是挂载，mount

#### 5.当前系统中安装了哪些包，如何查
```
rpm -qa | more  //分屏显示所有已安装的包
rpm -q vim-common //查看是否安装vim-common
```
#### 6.如何安装rpm包
先下载rpm包
```
rpm -i /mnt/Packages/apr-util-1.5.2-6.el7.x86_64.rpm
```
#### 7.如何卸载rpm包
```
rpm -e apr-util
```

## 二.yum包管理器
yum包可以解决rpm包安装的依赖关系
#### 1.yum源及国内镜像
- yum源：http://mirror.centos.org/centos/7/
- 国内镜像：https://opsx.alibaba.com/mirror
#### 2.如何修改yum源为国内镜像
```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup  //备份
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache //生成缓存
```
#### 3.yum命令
```
yum install vim-enhanced  //安装
yum remove vim //卸载
yum list //查看已安装的包
yum update //升级所有软件包
```
## 三.通过源码编译安装软件包
- 获取压缩包
```
wget https://openresty.org/download/openresty-1.15.8.1.tar.gz
```
- 解压压缩包
```
tar -zxf openresty-1.15.8.1.tar.gz
```
- 进入解压后的目录
```
cd openresty-1.15.8.1
```
- 执行./configure，并指定安装目录
```
./configure --prefix=/usr/local/openresty  //在这期间可能会报错，需要安装依赖包，可通过yum进行安装
```
- 执行make，将源码编译为可执行文件
```
make -j2 //-j2可以指定用2个逻辑cpu加快安装，也可直接make
```
- 执行make install，安装至/usr/local/openresty目录
```
make install
```
- 后期若想卸载，可直接删除目录：/usr/local/openresty

## 四.Linux内核升级
#### 1.查看当前内核版本
```
uname -r
```
#### 2.升级内核版本
```
yum install kernel-3.10.0
或 yum update
```
#### 3.源代码编译安装内核
- 安装依赖包
```
yum install gcc gcc-c++ make ncurses-devel openssl-devel elfutils-libelf-devel
```
- 下载并解压缩内核
```
下载：https://www.kernel.org
tar xvf linux-5.1.10.tar.xz -C /usr/src/kernels //
```
- 配置内核编译参数
```
cd /usr/src/kernels/linux-5.1.10/
make menuconfig | allyesconfig |allnoconfig
```
- 使用当前系统内核配置
```
cp /boot/config-kernelversion.platform /usr/src/kernels/linux-5.1.10/.config
```
- 编译
```
make -j2 all //all 表示对所有的编译选项都进行编译
```
- 安装内核
```
make modules_install
make install
```