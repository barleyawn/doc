# Nginx stream模块

> nginx从1.9.0开始，新增加了一个stream模块，用来实现四层协议的转发、代理或者负载均衡等。这完全就是抢HAproxy份额的节奏，鉴于nginx在7层负载均衡和web service上的成功，和nginx良好的框架，stream模块前景一片光明。

## Stream 模块编译

> stream模块默认没有编译到nginx， 编译nginx时候 ./configure –with-stream 即可


## Stream 用法

* 例： Rabbitmq负载均衡

```
upstream rabbitmq {
    server 192.168.1.122:5672 weight=5 max_fails=1 fail_timeout=30s;
    server 192.168.1.123:5672 weight=5 max_fails=1 fail_timeout=30s;
}

server{
    listen 5672;
    proxy_connect_timeout 30s;
    proxy_timeout 30s;
    proxy_pass rabbitmq;
}

```

用法跟 http 相似，均包含 upstram、server、listen等指令