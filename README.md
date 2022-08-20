## 目录

* [入门](#入门)
    * [安装](#安装)
    * [resty](#openresty-cli)
    * [load module](#加载-lua-模块)

* [LuaJIT & Lua](#luajit&lua)
    * [语法](lua/README.md)
    * [发展](#发展)
    * [优势](#优势)
    * [FFI](#FFI)
    * [NYI](#NYI)
    
* [性能优化](#性能优化)

* [性能分析](#性能分析)
    * [火焰图](#火焰图)
    


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


## LuaJIT & Lua

### 发展

标准 Lua 和 LuaJIT 是两回事儿，[LuaJIT](https://luajit.org/luajit.html) 只是兼容了 Lua 5.1 的语法，并对 Lua 5.2 和 5.3 做了选择性支持。

值得注意的是，OpenResty 并没有直接使用 LuaJIT 官方提供的 2.1.0-beta3 版本，而是在此基础上，维护了自己的 [LuaJIT 分支](https://github.com/openresty/luajit2) ，并扩展了很多独有的 API。

### 优势

其实标准 Lua 出于性能考虑，也内置了虚拟机，所以 Lua 代码并不是直接被解释执行的，而是先由 Lua 编译器编译为字节码（Byte Code），然后再由 Lua 虚拟机执行。

而 LuaJIT 的运行时环境，除了一个汇编实现的 Lua 解释器外，还有一个可以直接生成机器代码的 JIT 编译器。开始的时候，LuaJIT 和标准 Lua 一样，Lua 代码被编译为字节码，字节码被 LuaJIT 的解释器解释执行。

但不同的是，LuaJIT 的解释器会在执行字节码的同时，记录一些运行时的统计信息，比如每个 Lua 函数调用入口的实际运行次数，还有每个 Lua 循环的实际执行次数。
当这些次数超过某个随机的阈值时，便认为对应的 Lua 函数入口或者对应的 Lua 循环足够热，这时便会触发 JIT 编译器开始工作。

JIT 编译器会从热函数的入口或者热循环的某个位置开始，尝试编译对应的 Lua 代码路径。编译的过程，是把 LuaJIT 字节码先转换成 LuaJIT 自己定义的中间码（IR），
然后再生成针对目标体系结构的机器码。所以，所谓 LuaJIT 的性能优化，本质上就是让尽可能多的 Lua 代码可以被 JIT 编译器生成机器码，
而不是回退到 Lua 解释器的解释执行模式。明白了这个道理，你才能理解后面学到的 OpenResty 性能优化的本质。

### FFI

LuaJIT 除了兼容 Lua 5.1 的语法并支持 JIT 外，LuaJIT 还紧密结合了 FFI（Foreign Function Interface），
可以让你直接在 Lua 代码中调用外部的 C 函数和使用 C 的数据结构。

下面是一个最简单的例子：

```lua
local ffi = require("ffi")
ffi.cdef[[
int printf(const char *fmt, ...);
]]
ffi.C.printf("Hello %s!", "world")
```

短短这几行代码，就可以直接在 Lua 中调用 C 的 printf 函数，打印出 Hello world!。你可以使用 resty 命令来运行它，看下是否成功。

类似的，我们可以用 FFI 来调用 NGINX、OpenSSL 的 C 函数，来完成更多的功能。实际上，FFI 方式比传统的 Lua/C API 方式的性能更优，
这也是 lua-resty-core 项目存在的意义。

此外，出于性能方面的考虑，LuaJIT 还扩展了 table 的相关函数：table.new 和 table.clear。
这是两个在性能优化方面非常重要的函数，在 OpenResty 的 lua-resty 库中会被频繁使用。

### NYI

LuaJIT 的运行时环境，除了一个汇编实现的 Lua 解释器外，还有一个可以直接生成机器代码的 JIT 编译器。

LuaJIT 中 JIT 编译器的实现还不完善，有一些原语它还无法编译，当 JIT 编译器在当前代码路径上遇到它不支持的操作时，便会退回到解释器模式。
而 JIT 编译器不支持的这些原语，其实就是我们今天要讲的 NYI，全称为 Not Yet Implemented。LuaJIT 的官网上有这些 [NYI 的完整列表](http://wiki.luajit.org/NYI) 。

#### 如何检测 NYI？

LuaJIT 自带的 jit.dump 和 jit.v 模块，它们都可以打印出 JIT 编译器工作的过程。

可以先在 init_by_lua 中，添加以下两行代码：

```lua
local v = require "jit.v"
v.on("/tmp/jit.log")
```

然后，运行你自己的压力测试工具，或者跑几百个单元测试集，让 LuaJIT 足够热，触发 JIT 编译。这些都完成后，再来检查 /tmp/jit.log 的结果。

当然，这个方法相对比较繁琐，如果你想要简单验证的话， 使用 resty 就足够了。

```lua
$ resty -j v -e 'for i=1, 1000 do 
    local newstr, n, err = ngx.re.gsub("hello, world", "([a-z])[a-z]+", "[$0,$1]", "i") 
end'
[TRACE   1 regex.lua:1081 loop]
[TRACE   2 (1/10) regex.lua:1116 -> 1]
[TRACE   3 (1/21) regex.lua:1084 -> 1]
```

其中，resty 的 -j 就是和 LuaJIT 相关的选项；后面的值为 dump 和 v，就对应着开启 jit.dump 和 jit.v 模式。
在 jit.v 模块的输出中，每一行都是一个成功编译的 trace 对象。刚刚是一个能够被 JIT 的例子，而如果遇到 NYI 原语，输出里面就会指明 NYI，比如下面这个 pairs 的例子：


```lua
resty -j v -e 'local t = {}
for i=1,100 do
    t[i] = i
end

for i=1, 1000 do
    for j=1,1000 do
        for k,v in pairs(t) do
            local a = 0
        end
    end
end'
```

它就不能被 JIT，所以结果里，指明了第 8 行中有 NYI 原语。

```lua
[TRACE   1 t.lua:2 loop]
[TRACE --- t.lua:7 -- NYI: bytecode 72 at t.lua:8]
[TRACE --- t.lua:7 -- NYI: bytecode 72 at t.lua:8]
```



## 相关链接

[awesome-resty](https://github.com/bungle/awesome-resty)

[opm](https://github.com/openresty/opm)

[lua-nginx-module](https://github.com/openresty/lua-nginx-module)
