## 目录

* [入门](#入门)
    * [安装](#安装)
    * [OpenResty CLI](#openresty-cli)
    * [Load Lua](#加载-lua-模块)


## 入门

OpenResty 诞生于 2007 年，基于成熟的开源组件——NGINX 和 LuaJIT。

三大特性：`同步非阻塞`、`动态`、`详尽的文档和测试用例`。


### 安装

从 [官网](https://openresty.org/en/download.html) 获取安装包，使用源码安装的方式。

依赖包：

```shell
$ cat /etc/redhat-release
CentOS Linux release 7.8.2003 (Core)

$ yum -y install pcre-devel-8.32 openssl-devel-1.0.2k openssl-1.0.2k
```

源码编译安装：

```shell
$ wget https://openresty.org/download/openresty-1.11.2.1.tar.gz

$ tar zxvf openresty-1.11.2.1.tar.gz $$ cd openresty-1.11.2.1

$ ./configure  --with-debug  --with-cc-opt="-g3 -O2"  --with-ld-opt="-g3 -O2" --prefix=/usr/local/openresty

$ make install
```

验证：

```shell
$ /usr/local/openresty/bin/openresty -v
nginx version: openresty/1.11.2.1
```

### OpenResty CLI

安装完 OpenResty 后，默认就已经把 OpenResty 的 CLI：resty 安装好了。resty是个 1000 多行的 Perl 脚本，之前我们提到过，
OpenResty 的周边工具都是 Perl 编写的，这个是由 OpenResty 作者的技术偏好决定的。

```shell
$  /usr/local/openresty/bin/resty -e "ngx.say('hello world')"
hello world
```

resty 的功能很强大，更多的使用方式参考[官网](https://github.com/openresty/resty-cli) 。

下面使用 resty 创建一个 web 服务，访问并获取响应结果。

先来创建工作目录：

```shell
$ mkdir web
$ mkdir logs
$ mkdir conf
```

编写 nginx.conf 配置文件：

```shell
events {
    worker_connections 1024;
}

http {
    server {
        listen 8080;
        location / {
            content_by_lua '
                ngx.say("hello, world")
            ';
        }
    }
}
```

启动服务：

```shell
$ /usr/local/openresty/bin/openresty -p `pwd` -c conf/nginx.conf
```

使用 curl 命令，查看响应值：

```shell
$ curl -i 127.0.0.1:8080
HTTP/1.1 200 OK
Server: openresty/1.11.2.1
Date: Wed, 03 Aug 2022 04:17:28 GMT
Content-Type: text/plain
Transfer-Encoding: chunked
Connection: keep-alive

hello, world
```

### 加载 Lua 模块

在 web 的工作目录下，创建一个名为 lua 的目录，专门用来存放代码：

```shell
$ mkdir lua
$ cat lua/hello.lua
local _M = {}
local function run()
    ngx.say("hello, world")
end
_M.run = run
return _M
```

然后修改 nginx.conf 的配置，把 content_by_lua 改为 rewrite_by_lua_block：

```shell
events {
    worker_connections 1024;
}

http {
    lua_package_path "$prefix/lua/?.lua;;";

    server {
        listen 8080;
        location / {
             rewrite_by_lua_block {
                  require("hello").run()
             }
        }
    }
}
```

使用 curl 命令，查看响应值：

```shell
$ curl -i 127.0.0.1:8080
HTTP/1.1 200 OK
Server: openresty/1.11.2.1
Date: Thu, 04 Aug 2022 02:35:17 GMT
Content-Type: text/plain
Transfer-Encoding: chunked
Connection: keep-alive

hello, world
```

注意 `lua_package_path` 指令，用来设置 Lua 模块的查找路径，可以把 `lua_package_path` 设置为 `$prefix/lua/?.lua;;`，其中，
* `$prefix` 就是启动参数中的 `-p PATH`；
* `/lua/?.lua` 表示 lua 目录下所有以 .lua 作为后缀的文件；
* 最后的两个分号，则代表内置的代码搜索路径。


*** 

收录的第三方 restry 库 [awesome-resty](https://github.com/bungle/awesome-resty)

作者亲自操刀的网站类项目 [opm](https://github.com/openresty/opm)

核心模块 Lua nginx module [lua-nginx-module](https://github.com/openresty/lua-nginx-module)
