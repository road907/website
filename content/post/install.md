作为一个程序员， 当我们说安装软件的时候，到底在做什么呢？ 比如，我们如果要在一台Debian系统上，安装Kubelet， 常用操作是：

 curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add cat <<EOF >/etc/apt/sources.list.d/kubernetes.list deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main EOF apt-get update apt-get install kubelet 那么，这篇文章将梳理这几条命令背后做了什么， 也就是Debian软件管理的大概了。

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyMTQ4MTQwODNdfQ==
-->