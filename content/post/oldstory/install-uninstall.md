---
title: "计算机考古之软件安装与卸载"
date: 2019-06-16T18:17:07+08:00
draft: true
---


作为一个程序员， 当我们说安装软件的时候，到底在做什么呢？
比如，我们如果要在一台Debian系统上，安装Kubelet， 常用操作是：

    curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add 
    cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
    EOF  
    apt-get update
    apt-get install  kubelet 

那么，这篇文章将梳理这几条命令背后做了什么， 也就是Debian软件管理的大概了。
此问题，可分为四个层次：
<center>![debian 包管理层次](/layers.jpg)</center>

## Filesystem Hierarchy Standard

文件系统分层标准， 即类UNIX操作系统上，各个目录和文件的使用标准。 例如： 

- /user 原则上放置共享、只读数据 。  比如 /user/bin  /user/lib
- /var 原则上放置可以变数据。比如 /var/log   /var/cache

为什么关心这个呢？ 因为操作系统本身是个Software， Debian Software 是对 "OS Software"的扩展。所谓的安装最重要的就是把
软件相关文件plug到操作系统的文件系统中，而卸载是要把这些文件删除掉。

## Software Package

Debian 的软件包即一个包含了安装内容、安装信息的归档文件。 
可以用 sudo apt-get download kubelet ， 把要安装的.deb文件下载下来。然后打开这个归档，可以看到如下结构：

<center>![debian package content](/deb-files.jpeg)</center>

- debian-binay:  只有一行内容， 指示这个.deb包格式的版本号， 目前为2.0
- data.tar.xz:   为安装的内容， 可见他是按上述FHS来做的， 包含一个可执行文件kubelet 和  一个systemd配置。
- control.tar.gz:  包含包控制信息。

详细看下包控制信息：

- control:  比较重要的信息是版本号、架构、依赖。打开看：
```
Package: kubelet
Version: 1.14.2-00
Architecture: amd64
Maintainer: Kubernetes Authors <kubernetes-dev+release@googlegroups.com>
Installed-Size: 124994
Depends: iptables (>= 1.4.21), kubernetes-cni (= 0.7.5), iproute2, socat, util-linux, mount, ebtables, ethtool, conntrack, init-system-helpers (>= 1.18~)
Section: misc
Priority: optional
Homepage: https://kubernetes.io
Description: Kubernetes Node Agent
```
- md5sums:  顾名思义, 包体的哈希值，用于校验包的安全性
- prerm\postinst\postrm:   为可执行脚本，在软件生命周期的不同阶段触发执行。postinst 是软件安装完后，要执行的脚本， 打开看有如下内容：
```
if [ -d /run/systemd/system ]; then
    systemctl --system daemon-reload >/dev/null || true
    deb-systemd-invoke start kubelet.service >/dev/null || true
fi
```
意思就是安装后，加入systemd并启动啦。

## Local Software Management

dpkg 是debian用来管理本地软件的。`apt-get install`在本地安装时依赖的是dpkg。 比如用`dpk -l`可查看系统上所有debian软件。
`/var/lib/dpkg/` 目录可认为是dpkg的数据库，记录所有软件装填。
`/var/lib/dpkg/info` 存放了所有安装软件的控制信息。 
比如现在我们使用”sudo apt-get instal kubelet“,  即可看到如下文件:
```
-rw-r--r-- 1 root root  108 Jun  1 17:11 kubelet.list
-rw-r--r-- 1 root root  119 May 17 03:29 kubelet.md5sums
-rwxr-xr-x 1 root root 1938 May 17 03:29 kubelet.postinst
-rwxr-xr-x 1 root root  590 May 17 03:29 kubelet.postrm
-rwxr-xr-x 1 root root  184 May 17 03:29 kubelet.prerm
```
打开kubelet.list看：
```
/.
/lib
/lib/systemd
/lib/systemd/system
/lib/systemd/system/kubelet.service
/usr
/usr/bin
/usr/bin/kubelet
```
即包含了所有安装内容，那么卸载的时候就会删除这些文件，相关脚本也会执行，比如postrm。

## Distributed Software Management

分布式软件管理是完整的一套机制，可想到包括： 软件仓库、版本管理、依赖管理。
APT只是Debian分布式软件管理的客户端。
<center>![distributed pkg mgm](/distributed-packages.jpg)</center>

如上， 远程仓库核心的两份数据是，它托管的所有应用包和应用包的索引。
apt 会在本地通过sources.list文件配置可用仓库的地址。
`apt-get update` 即拉取索引信息， apt即可知道每个仓库托管了什么应用。
`apt-get install` 就是从响应的仓库拉取应用并且调用dpkg安装了。

以阿里云的镜像仓库`https://mirrors.aliyun.com/kubernetes`为例，其目录结构大概如下：
<center>![ali-repos](/ali-pkg-repos.png)</center>

- dists:  内部存储索引信息，不过不是把所以索引放到一个packages.gz文件，而是又有两个层次。
- pool:   存储所有应用包啦。
- doc:  文档、秘钥等。 

与APT相关的重要目录有：

- `/etc/apt/sources.list`  或者  `/etc/apt/sources.list.d/`  配置仓库地址。
- `/var/lib/apt/lists/` ： `apt-get update` 拉取所有仓库的索引信息会放到这。 
- `/var/cache/apt/archives/` ： 存放所有从仓库下载的包。

看下`sources.list`仓库地址是怎么配的，假设有如下下配置：
```
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main contrib
```
apt 可以拉取的所有信息包括两个：
```
https://mirrors.aliyun.com/kubernetes/apt/dists/kubernetes-xenial/main
https://mirrors.aliyun.com/kubernetes/apt/dists/kubernetes-xenial/contrib
```
即 main contrib 为并列的概念，在Debian设计中，他们叫"Archive Area"。

回到一开始的问题，逐行分析APT干了什么：
```
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add 
```
因为要添加一个软件仓库， 处于安全考虑，需要添加对应的秘钥，用于验证下载的包没有经过篡改。 
```
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF 
```
即添加仓库地址
```
apt-get update
```
拉取索引。执行完成后，到`/var/lib/apt/lists/` 下看，有如下索引信息：
```
mirrors.aliyun.com_kubernetes_apt_dists_kubernetes-xenial_main_binary-amd64_Packages
```
，打开看, 是各种包的索引信息：
<center>![apt-indexes](/apt-pkg-indexes.png)</center>
`apt-get install kubelet` , 根据索引信息的filename拉取应用包并安装。

参考：

1. https://en.wikipedia.org/wiki/Deb_(file_format)
2. https://en.wikipedia.org/wiki/Dpkg
3. https://en.wikipedia.org/wiki/APT_(Package_Manager)
4. http://www.pathname.com/fhs/pub/fhs-2.3.pdf
5. https://www.debian.org/doc/debian-policy/index.html
6. https://www.jianshu.com/p/6432015c52a6
7. http://cn.linux.vbird.org/linux_basic/0520rpm_and_srpm.php