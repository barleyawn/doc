# Nginx 编译安装


## configure 
> 在运行configure脚本命令的时候，可能需要开启一些开关选项，开关选项可能因为下载的版本不同有差异，使用./configure --help 列出有效的开关选项：


| 路径选项 | 用法 | 默认值 |
| :-----: | :----: | :----: |
| --prefix | nginx安装位置 | /usr/local/openresty |
| --sbin-path | nginx二进制执行文件路径 | /sbin/nginx |
| --conf-path | 配置文件存放位置 | /conf/nginx.conf |
| --error-log-path | 错误日志文件存放位置 | /logs/error.log |
| --pid-path | pid文件路径 | /logs/nginx.pid |
| --lock-path | 锁文件存放路径 | /logs/nginx.log |
| –-with-perl_modules_path | perl模块路径 |  |
| –-with-perl | perl执行文件路径 |  |
| –-http-log-path | 访问日志文件路径 | /logs/access.log |
| –-http-client-body-temp-path | 客户端请求产生的临时文件路径 | /client_body_temp |
| –-http-proxy-temp-path | 代理存储临时文件路径	 | /proxy_temp_path |
| –-http-proxy-temp-path | 代理存储临时文件路径	 | /proxy_temp_path |
| –-http-fastcgi-temp-path | HTTP FastCGI模块使用的临时文件路径	 |  |
| –-builddir | 创建应用程序位置	 |  |

> 先决条件选项: 先决条件的格式有库文件和二进制文件，在配置之前需要先安装这些依赖软件，有时即使这些软件已经安装在系统中，可能有时配置脚本还是找不到它们的位置，因此可以使用开关项指出它们的路径。

| 编译选项 | 用法 | 
| :-----: | :----: | 
| --with-cc | 指定C编译器位置 | 
| --with-cc | 指定备用C预处理位置 | 
| --with-cc-opt | 定义额外选项，然后在命令行传递给C编译器 | 
| --with-ld-opt | 定义额外选项，然后在命令行传递给C连接器 | 
| --with-cpu-opt | 指定不同的目标处理器结构，可以是下列值：pentium、pentiumpro、ppc64等 |

| PCRE选项 | 用法 | 
| :-----: | :----: | 
| --without-pcre | 不使用PCRE库 | 
| --with-pcre | 强制使用pcre | 
| --with-pcre=: | 指定pcre源代码(注意不是安装目录) | 
| --with-pcre-opt | 用于建立PCRE库的额外选项 | 