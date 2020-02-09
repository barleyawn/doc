# Nginx安装的准备工作

## 1 Linux操作系统

由于Linux具有免费、使用广泛、商业支持越来越完善等特点，本次将主要针对Linux上运行Nginx。

> 需要一个内核 Linux2.6及以上版本的操作系统， 因为Linux2.6及以上的内核才支持[epoll](),在Linux系统上使用[select]()或[poll]()来解决事件的多路复用， 是无法解决高并发压力问题的。

本次使用的系统是 阿里云的 Centos 7,通过 uname -a 命令查询 Linux内核版本
```
uname -a
Linux iZ2ze7ytqh359wvsikz5c7Z 3.10.0-1062.1.2.el7.x86_64 #1 SMP Mon Sep 30 14:19:46 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```
结果表示 内核版本是 3.10.0

## 2 必备环境及软件

1. [GCC]()编译器

GCC（GNU Compiler Collection）可用来编译C语言程序。Nginx不会直接提供二进制可执行程序，所以我们需要对Nginx的源码进行编译安装。我们可以使用最简单的yum方式安装GCC，例如：

    yum install -y gcc 


2. PCRE库

PCRE（Perl Compatible Regular Expressions，Perl兼容正则表达式）是由Philip Hazel开发的函数库，目前为很多软件所使用，该库支持正则表达式。它由RegEx演化而来，实际上，Perl正则表达式也是源自于Henry Spencer写的RegEx。

如果我们在配置文件nginx.conf里使用了正则表达式，那么在编译Nginx时就必须把PCRE库编译进Nginx，因为Nginx的HTTP模块要靠它来解析正则表达式。当然，如果你确认不会使用正则表达式，就不必安装它。其yum安装方式如下：

    yum install -y pcre pcre-devel 

> pcre-devel是使用PCRE做二次开发时所需要的开发库，包括头文件等，这也是编译Nginx所必须使用的。

3. zlib库

zlib库用于对HTTP包的内容做gzip格式的压缩，如果我们在nginx.conf里配置了gzip on，并指定对于某些类型（content-type）的HTTP响应使用gzip来进行压缩以减少网络传输量，那么，在编译时就必须把zlib编译进Nginx。其yum安装方式如下：

    yum install -y zlib zlib-devel 

4. OpenSSL开发库

如果我们的服务器不只是要支持HTTP，还需要在更安全的SSL协议上传输HTTP，那么就需要拥有OpenSSL了。另外，如果我们想使用MD5、SHA1等散列函数，那么也需要安装它。其yum安装方式如下：

    yum install -y openssl openssl-devel 

## 3. 磁盘目录

* Nginx源代码存放目录
* Nginx编译阶段产生的中间文件存放目录
* 部署目录，默认情况下， 是 /usr/local/nginx
* 日志文件存放目录

## 4. 内核参数优化

因为Linux默认的内核参数考虑的时最通用的场景，但这不符合用于支持高并发访问的web服务器，所以需要根据业务特点（将Nginx作为静态web内容服务器、反向代理服务器或者是提供图片缩略图实时压缩图片的服务器）来修改Linux内核参数，使得Nginx可以拥有更高的性能。
需要修改`/etc/sysctl.conf`来更改内核参数
最常用的配置：

```
fs.file_max = 999999
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.ip_local_port_range = 1024 61000
net.ipv4.tcp_rmen = 4096 32768 262142
net.ipv4.tcp_wmen = 4096 32768 262142
net.ipv4.tcp_max_syn_backlog = 1024
net.core.rmen_default = 262144
net.core.wmen_default = 262144
net.core.rmen_max = 2097152
net.core.wmen_max = 2097152
net.ipv4.tcp_syncookies = 1
```

然后执行 `sysctl -p` 命令，使上述修改生效。
其中一些参数的意义解释如下:

* file_max：表示进程（例如一个worker进程）可以同时打开的最大句柄数，这个参数直接限制最大并发连接数，需要根据实际情况配置。
* tcp_tw_reuse：这个参数设置为1，表示允许将TIME-WAIT状态socket重新用于新的TCP连接，这对于服务器来说很有意义，因为服务器上总会有大量TIME-WAIT状态的连接。
* tcp_keepalive_time：这个参数表示当keepalive启用时，TCP发送keepalive消息的频度。默认是2小时，若将其设置得小一点，可以更快地清理无效的连接。
* tcp_fin_timeout ：这个参数表示当服务器主动关闭连接时，socket保持在FIN-WAIT-2状态的最大时间。
* tcp_max_tw_buckets：这个参数表示操作系统允许TIME-WAIT套接字数量的最大值，如果超过这个数字，TIME-WAIT套接字将立刻被清除并打印警告消息。该参数默认是180000，过多的TME-WAIT套接字会使web服务器变慢。
* tcp_max_syn_backlog：这个参数表示TCP三次握手建立阶段接收SYN请求队列的最大长度，默认为1024，将其设置得大一些可以使出现nginx繁忙来不及accept新链接的情况时，Linux不至于丢失客户端发起的连接请求。
* ip_local_port_range：这个参数定义了在UDP和TCP连接中本地（不包括连接的远端）端口就得取值范围。
* tcp_rmen：这个参数定义了TCP接收缓存（用于TCP接收滑动窗口）的最小值、默认值、最大值。
* tcp_wmen：这个参数定义了TCP发送缓存（用于TCP发送滑动窗口）的最小值、默认值、最大值。
* rmen_default：这个参数表示内核套接字接收缓存区默认的大小。
* wmen_default：这个参数表示内核套接字发送缓存区默认的大小。
* rmen_max：这个参数表示内核套接字接收缓存区最大大小。
* wmen_max：这个参数表示内核套接字发送缓存区最小大小。
* tcp_syncookies：该参数与性能无关，用于解决TCP的SYN攻击。