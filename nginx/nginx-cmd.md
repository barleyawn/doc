# Nginx的命令行控制

在Linux中，需要使用命令行来控制Nginx服务器的启动和停止、重载配置文件、回滚日志文件、平滑升级等行为。
> 默认情况下，Nginx安装在 `/usr/local/nginx/`中

1. 默认方式启动

`/usr/local/nginx/sbin/nginx`

这时默认加载的 配置文件： `/usr/local/nginx/conf/nginx.conf`

2. 指定配置文件方式启动

`/usr/local/nginx/sbin/nginx -c /tmp/nginx.conf`

3. 指定Nginx安装的目录

`/usr/local/nginx/sbin/nginx -p /usr/local/nginx/`

4. 指定全局配置项的启动方式

`/usr/local/nginx/sbin/nginx -g 'pid /var/nginx/nginx.pid;'`

上述命令，会将`pid`文件写到 `/var/nginx/nginx.pid`中

> -g 参数的约束条件：
* 指定的配置项不能与默认路径下的 `nginx.conf` 中的配置项相冲突，否则无法启动。 如果执行前面的命令，不能在在`nginx.conf`中出现配置项： `pid logs/nginx.pid;`。

* 以 -g 方式启动的Nginx服务执行其他命令时，需要把 -g 参数也带上， 否则可能出现配置项不匹配的情形。

上述命令，在停止Nginx服务时，需要执行：
`/usr/local/nginx/sbin/nginx -g 'pid /var/nginx/nginx.pid;' -s stop`

5. 测试配置项信息是否正确

`/usr/local/nginx/sbin/nginx -t`

6. 测试配置项信息时不输出信息

`/usr/local/nginx/sbin/nginx -t -q`

7. 显示版本信息

`/usr/local/nginx/sbin/nginx -v`

8. 显示编译阶段的参数

`/usr/local/nginx/sbin/nginx -V`

9. 快速地停止服务

`/usr/local/nginx/sbin/nginx -s stop`

> 使用` -s stop`可以强制停止Nginx服务。 -s 参数是告诉Nginx程序向正在运行的Nginx服务发送信号量，Nginx程序通过nginx.pid文件中得到master进程的进程ID,再向运行中的master进程发送TERM信号来快速地关闭Nginx服务。

* 实际上，也可以通过`kill`命令直接向nginx master进程发送`TERM`或`INT`信号。
```
# 查看nginx进程
>  ps -ef | grep nginx
>  kill -s SIGTERM pid
>  kill -s SIGINT pid
```
10. "优雅"地停止服务
> 如果希望Nginx服务可以正常处理完当前所有请求再停止服务，可以使用 ` -s quit`来停止服务。该命令与 ` -s stop`是有区别的。


> 当 ` -s stop`时，worker进程与master进程在收到信号后会立刻跳出循环，退出进程。 而 ` -s quit`时， 首先会关闭监听端口，并停止接收新的连接，然后处理完当前所有请求后，再退出进程。



`/usr/local/nginx/sbin/nginx -s quit`


* 实际上，也可以通过`kill`命令直接向nginx master进程发送`QUIT`信号。
```
>  kill -s SIGQUIT pid
```

* 如果希望优雅的停止某个worker进程，可以通过以下命令：

`kill -s SIGWINCH <nginx worker pid>`



11. 重载配置项并生效

`/usr/local/nginx/sbin/nginx -s reload`

> 该命令会先检查新的配置项是否有误，如果全部正确就以“优雅“的方式停止服务，再重新启动服务。
* 实际上，也可以通过`kill`命令直接向nginx master进程发送`HUP`信号。
```
kill -s SIGHUP <nginx master pid>
```

12. 日志文件回滚

`/usr/local/nginx/sbin/nginx -s reload`

