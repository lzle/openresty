## 目录

* [入门](#入门)
    * [安装](#安装)
    * [resty](#openresty-cli)
    * [load module](#加载-lua-模块)
* [LuaJIT & Lua](#luajit--lua)
    * [语法](lua/README.md)
    * [发展](#发展)
    * [性能](#优势)
    * [FFI](#FFI)
    * [NYI](#NYI)
* [性能优化](#性能优化)
    * [阻塞](#阻塞)
    * [字符串](#字符串)
    * [table](#table)
* [API](#API)
    * [ngx.ctx](#ngxctx)
    * [ngx.shared.DICT](#ngxsharedDICT)
    * [ngx.var](#ngxvar)
    * [ngx.socket.tcp](#ngxsockettcp)
    * [ngx.re.split](#ngxresplit)
    * [ngx.re.match](#ngxrematch)
    * [ngx.re.find](#ngxresfind)
    * [ngx.re.gmatch](#ngxregmatch)
    * [ngx.re.sub](#ngxresub)
    * [ngx.re.gsub](#ngxregsub)
    * [ngx.semaphore](#ngxsemaphore)
    * [ngx.thread.spawn](#ngxthreadspawn)
    * [ngx.thread.wait](#ngxthreadwait)
    * [ngx.thread.kill](#ngxthreadkill)
* [Lua Resty](#Resty)
    * [lua-resty-core](#lua-resty-core)
    * [lua-resty-string](#lua-resty-string)
* [Test::Nginx](#testnginx)
    * [Test](#Test)
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
$ wget https://openresty.org/download/openresty-1.19.9.1.tar.gz

$ tar zxvf openresty-1.19.9.1.tar.gz $$ cd openresty-1.19.9.1

$ ./configure  --with-debug  --with-cc-opt="-g3 -O2"  --with-ld-opt="-g3 -O2" --prefix=/usr/local/openresty

$ make install
```

验证：

```shell
$ /usr/local/openresty/bin/openresty -v
nginx version: openresty/1.19.9.1
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

## 性能优化

在 OpenResty 2019 年 5 月份发布的 1.15.8.1 版本前 lua-resty-core 默认是不开启的，而这不仅会带来性能损失，更严重的是会造成潜在的 bug。所以，我强烈推荐还在使用历史版本的用户，都手动开启 lua-resty-core。你只需要在 init_by_lua 阶段，增加一行代码就可以了：

```
require "resty.core"
```

在 OpenResty 中，我们总是优先使用 OpenResty 的 API，然后是 LuaJIT 的 API，使用 Lua 库则需要慎之又慎。

### 阻塞

重要原则：避免使用阻塞函数。

OpenResty 之所以可以保持很高的性能，简单来说，是因为它借用了 Nginx 的事件处理和 Lua 的协程机制，所以：

* 在遇到网络 I/O 等需要等待返回才能继续的操作时，就会先调用 Lua 协程的 yield 把自己挂起，然后在 Nginx 中注册回调； 
  
* 在 I/O 操作完成（也可能是超时或者出错）后，由 Nginx 回调 resume，来唤醒 Lua 协程。

在这个处理流程中，如果没有使用 cosocket 这种非阻塞的方式，而是用阻塞的函数来处理 I/O，那么 LuaJIT 就不会把控制权交给 Nginx 的事件循环。
这就会导致，其他的请求要一直排队等待阻塞的事件处理完，才会得到响应。

#### 1、执行外部命令

很多情况下需要调用外部的命令和工具，来辅助完成一些操作：

```lua
os.execute("kill -HUP " .. pid) 

os.execute(" cp test.exe /tmp ")

os.execute(" openssl genrsa -des3 -out private.pem 2048 ")
```

表面上看，os.execute 是 Lua 的内置函数，但是在 OpenResty 的环境中，os.execute 会阻塞当前请求。
所以，如果这个命令的执行时间特别短，那么影响还不是很大；可如果这个命令，需要执行几百毫秒甚至几秒钟的时间，那么性能就会有急剧的下降。

方案一： 如果有 FFI 库可以使用，那么我们就优先使用 FFI 的方式来调用。

比如，上面我们是用 OpenSSL 的命令行来生成密钥，就可以改为，用 FFI 调用 OpenSSL 的 C 函数的方式来绕过。而对于杀掉某个进程的示例，
你可以使用 lua-resty-signal 这个 OpenResty 自带的库，来非阻塞地解决。代码实现如下，当然，这里的lua-resty-signal ，其实也是用 FFI 去调用系统函数来解决的。

```lua
local resty_signal = require "resty.signal"
local pid = 12345


local ok, err = resty_signal.kill(pid, "KILL")
```

方案二：使用基于 ngx.pipe 的 lua-resty-shell 库。

正如之前介绍过的一样，你可以在 shell.run 中运行你自己的命令，它就是一个非阻塞的操作：

```lua
$ resty -e 'local shell = require "resty.shell"
local ok, stdout, stderr, reason, status =
    shell.run([[echo "hello, world"]])
    ngx.say(stdout) '
```

#### 2、磁盘 I/O

ngx.log 本身就是一个代价不小的函数调用，缺点在于，你不能频繁地去调用它。即使有缓冲区，大量而频繁的磁盘写入，也会严重地影响性能。

可以把日志发送到远端的日志服务器上，这样就可以用 cosocket 来完成非阻塞的网络通信了，也就是把阻塞的磁盘 I/O 丢给日志服务，不要阻塞对外的服务。
你可以使用 lua-resty-logger-socket ，来完成这样的工作：

```lua
local logger = require "resty.logger.socket"
if not logger.initted() then
    local ok, err = logger.init{
        host = 'xxx',
        port = 1234,
        flush_limit = 1234,
        drop_limit = 5678,
    }
local msg = "foo"
local bytes, err = logger.log(msg)
```

#### 3、luasocket

最后，我们来说说 luasocket ，它也是容易被开发者用到的一个 Lua 内置库，经常有人分不清 luasocket 和 OpenResty 提供的 cosocket。
luasocket 也可以完成网络通信的功能，但它并没有非阻塞的优势。如果你使用了 luasocket，那么性能也会急剧下降。

另外，[lua-resty-socket](https://github.com/thibaultcha/lua-resty-socket/) 其实就是一个二次封装的开源库，它做到了 luasocket 和 cosocket 的兼容。


### 字符串

在 Lua 中，字符串是不可变的。并不是说字符串不能做拼接、修改等操作，而是当进行这些操作时，会产生新的字符串对象。

要避免产生中间的无用数据:

```shell
$ resty -e 'local begin = ngx.now()
local s = ""
-- for 循环，使用 .. 进行字符串拼接
for i = 1, 100000 do
    s = s .. "a"
end
ngx.update_time()
print(ngx.now() - begin)
'
```

更好的方式是采用下面这种方式，减少了临时字符串的产生：

```shell
$ resty -e 'local begin = ngx.now()
local t = {}
-- for 循环，使用数组来保存字符串，自己维护数组的长度
for i = 1, 100000 do
    t[i] = "a"
end
local s =  table.concat(t, "")
ngx.update_time()
print(ngx.now() - begin)
'
```

在 ngx.say、ngx.print 、ngx.log、cosocket:send 等这些可能接受大量字符串的 API 中，它不仅接受 string 作为参数，也同时接受 table 作为参数。

### table

#### 1、优先 LuaJIT APIs

The following APIs can be JIT compiled.

**① table.isempty**

Usage:

```lua
local isempty = require "table.isempty"

print(isempty({}))  -- true
print(isempty({nil, dog = nil}))  -- true
print(isempty({"a", "b"}))  -- false
print(isempty({nil, 3}))  -- false
print(isempty({cat = 3}))  -- false
```

**② table.isarray**

Usage:

```lua
local isarray = require "table.isarray"

print(isarray{"a", true, 3.14})  -- true
print(isarray{dog = 3})  -- false
print(isarray{})  -- true
```

**③ table.nkeys**

Usage:

```lua
local nkeys = require "table.nkeys"

print(nkeys({}))  -- 0
print(nkeys({ "a", nil, "b" }))  -- 2
print(nkeys({ dog = 3, cat = 4, bird = nil }))  -- 2
print(nkeys({ "a", dog = 3, cat = 4 }))  -- 3
```

**④ table.clone**

Usage:

```lua
local clone = require "table.clone"

local x = {x=12, y={5, 6, 7}}
local y = clone(x)
... use y ...
```

Deep cloning is planned to be supported by adding true as a second argument.


#### 2、预先生成数组

每次新增和删除数组元素的时候，都会涉及到数组的空间分配、resize 和 rehash。

LuaJIT 中的 table.new(narray, nhash) 函数，会预先分配好指定的数组和哈希的空间大小，而不是在插入元素时自增长，这也是它的两个参数 narray 和 nhash 的含义。

```lua
local new_tab = require "table.new"
local t = new_tab(100, 0)
for i = 1, 100 do
    t[i] = i
end
```

#### 3、自己计算下标

向 table 里面增加元素，最直接的方法，就是调用 table.insert 这个函数来插入元素：

```lua
local new_tab = require "table.new"
local t = new_tab(100, 0)
for i = 1, 100 do
  table.insert(t, i)
end
```

或者是先获取当前数组的长度，通过下标的方式来插入元素：

```shell

local new_tab = require "table.new"
local t = new_tab(100, 0)
for i = 1, 100 do
  t[#t + 1] = i
end
```

需要注意的是这两个操作都是 O(n) 的时间复杂度，在热代码中性能不容乐观，并且数组越大时，性能也会越低。

另外，table.getn 并不是 O(1) 的时间复杂度，而是 O(n)。

#### 4、循环使用

table.clear 函数会把数组中的所有数据清空，但数组的大小不会变。

```shell

local ok, clear_tab = pcall(require, "table.clear")
if not ok then
    clear_tab = function (tab)
    for k, _ in pairs(tab) do
    tab[k] = nil
    end
end
end
```

table.clear 函数实际上就是把每一个元素都置为了 nil。

#### 5、table 池

还可以用缓存池的方式来保存多个 table，以便随用随取，官方提供的 lua-tablepool 正是出于这个目的。

```shell
local tablepool = require "tablepool"
local tablepool_fetch = tablepool.fetch
local tablepool_release = tablepool.release


local pool_name = "some_tag" 
local function do_sth()
     local t = tablepool_fetch(pool_name, 10, 0)
     -- -- using t for some purposes
    tablepool_release(pool_name, t) 
end
```

注意不要因此滥用 tablepool, 在实际项目中的使用并不多。


## API

[Nginx API for Lua](https://github.com/openresty/lua-nginx-module/#nginx-api-for-lua)

### ngx.ctx

该表可用于存储每个请求的 Lua 上下文数据，并且具有与当前 `request` 有相同的生命周期。

子请求有自己独立的 `ngx.ctx` 数据，与父请求的 `ngx.ctx` 互不影响。

```lua
location /sub {
    content_by_lua_block {
        ngx.say("sub pre: ", ngx.ctx.blah)
        ngx.ctx.blah = 32
        ngx.say("sub post: ", ngx.ctx.blah)
    }
}

location /main {
    content_by_lua_block {
        ngx.ctx.blah = 73
        ngx.say("main pre: ", ngx.ctx.blah)
        local res = ngx.location.capture("/sub")
        ngx.print(res.body)
        ngx.say("main post: ", ngx.ctx.blah)
    }
}
```

请求 `GET /main` 将会输出：

```shell
main pre: 73
sub pre: nil
sub post: 32
main post: 73
```

`ngx.ctx` 查找需要相对昂贵的元方法调用，而且它比通过您自己的函数参数显式传递每个请求的数据要慢得多。
所以不要滥用这个 API 来保存你自己的函数参数，因为它通常会对性能产生相当大的影响。

永远不要在函数外执行 `local ngx.ctx` 进行模块级别的共享，比如下面不好的示例：

```lua
-- mymodule.lua
local _M = {}

-- the following line is bad since ngx.ctx is a per-request
-- data while this <code>ctx</code> variable is on the Lua module level
-- and thus is per-nginx-worker.
local ctx = ngx.ctx

function _M.main()
    ctx.foo = "bar"
end

return _M
```

建议使用下面的方式：

```shell
-- mymodule.lua
local _M = {}

function _M.main(ctx)
    ctx.foo = "bar"
end

return _M
```

### ngx.shared.DICT

进程间共享数据的首选，所有操作都是原子的，一般用来做数据的缓存，减少接口调用的次数，如 `acid.cache` 模块。

支持的方法：
* get
* get_stale
* set
* safe_set
* add
* safe_add
* replace
* delete
* incr
* lpush
* rpush
* lpop
* rpop
* llen
* ttl
* expire
* flush_all
* flush_expired
* get_keys
* capacity
* free_space

[官网示例](https://github.com/openresty/lua-nginx-module#ngxshareddict) 


### ngx.var

`ngx.var` 是 nginx 内置变量，基本上每个 `ngx.var` 的变量都有相应的 `ngx.req` 方法。
例如 `ngx.var.request_method` 也可以通过 `ngx.req.get_method` 调用获得，那为啥还要再封装 `ngx.req` 接口呢？

这其实是很多方面因素的综合考虑结果：

* 首先是对性能的考虑。ngx.var 的效率不高，不建议反复读取；
  
* 也有对程序友好的考虑，ngx.var 返回的是字符串，而非 Lua 对象，遇到获取 args 这种可能返回多个值的情况，就不好处理了；
  
* 另外是对灵活性的考虑，绝大部分的 ngx.var 是只读的，只有很少数的变量是可写的，比如 $args 和 limit_rate，
  可很多时候，我们会有修改 method、URI 和 args 的需求。

  
### ngx.socket.tcp

创建 TCP 的 `cosocket` 对象，`cosocket` 是各种 lua-resty-* 非阻塞库的基础。

支持的方法：

* bind
* connect
* setclientcert
* sslhandshake
* send
* receive
* close
* settimeout
* settimeouts
* setoption
* receiveany
* receiveuntil
* setkeepalive
* getreusedtimes

[官网示例](https://github.com/openresty/lua-nginx-module/#ngxsockettcp) 


### ngx.re.split

字符串切割是很常见的功能，OpenResty 也提供了对应的 API，很多开发者都找不到这样的函数，只能选择自己手写。

为什么呢？其实， ngx.re.split 这个 API 并不在 lua-nginx-module 中，而是在 lua-resty-core 里面；

```lua
local ngx_re = require "ngx.re"

local res, err = ngx_re.split("a,b,c,d", "(,)")
-- res is now {"a", ",", "b", ",", "c", ",", "d"}
```

### ngx.re.match

Matches the subject string using the Perl compatible regular expression regex with the optional options.

```lua
local m, err = ngx.re.match("hello, 1234", "([0-9])[0-9]+")
 -- m[0] == "1234"
 -- m[1] == "1"
```

[官网示例](https://github.com/openresty/lua-nginx-module#ngxrematch) 


### ngx.re.find

Similar to ngx.re.match but only returns the beginning index (from) and end index (to) of the matched substring. The returned indexes are 1-based and can be fed directly into the string.sub API function to obtain the matched substring.

```lua
local s = "hello, 1234"
local from, to, err = ngx.re.find(s, "([0-9]+)", "jo")
if from then
    ngx.say("from: ", from)
    ngx.say("to: ", to)
    ngx.say("matched: ", string.sub(s, from, to))
else
    if err then
        ngx.say("error: ", err)
        return
    end
    ngx.say("not matched!")
end
```

[官网示例](https://github.com/openresty/lua-nginx-module#ngxrefind) 

### ngx.re.gmatch

Similar to ngx.re.match, but returns a Lua iterator instead, so as to let the user programmer iterate all the matches over the <subject> string argument with the PCRE regex.

```lua
local iterator, err = ngx.re.gmatch("hello, world!", "([a-z]+)", "i")
if not iterator then
    ngx.log(ngx.ERR, "error: ", err)
    return
end

local m
m, err = iterator()    -- m[0] == m[1] == "hello"
if err then
    ngx.log(ngx.ERR, "error: ", err)
    return
end

m, err = iterator()    -- m[0] == m[1] == "world"
if err then
    ngx.log(ngx.ERR, "error: ", err)
    return
end

m, err = iterator()    -- m == nil
if err then
    ngx.log(ngx.ERR, "error: ", err)
    return
end
```

[官网示例](https://github.com/openresty/lua-nginx-module#ngxregmatch) 


### ngx.re.sub

Substitutes the first match of the Perl compatible regular expression regex on the subject argument string with the string or function argument replace. The optional options argument has exactly the same meaning as in ngx.re.match.

```lua
local newstr, n, err = ngx.re.sub("hello, 1234", "([0-9])[0-9]", "[$0][$1]")
if not newstr then
    ngx.log(ngx.ERR, "error: ", err)
    return
end

-- newstr == "hello, [12][1]34"
-- n == 1
```

[官网示例](https://github.com/openresty/lua-nginx-module#ngxresub) 

### ngx.re.gsub

Just like ngx.re.sub, but does global substitution.

```lua
local newstr, n, err = ngx.re.gsub("hello, world", "([a-z])[a-z]+", "[$0,$1]", "i")
if not newstr then
    ngx.log(ngx.ERR, "error: ", err)
    return
end

-- newstr == "[hello,h], [world,w]"
-- n == 2
```

[官网示例](https://github.com/openresty/lua-nginx-module#ngxregsub) 


### ngx.semaphore

1、在相同的 `context` 线程中同步。

```
location = /t {
    content_by_lua_block {
        local semaphore = require "ngx.semaphore"
        local sema = semaphore.new()

        local function handler()
            ngx.say("sub thread: waiting on sema...")

            local ok, err = sema:wait(1)  -- wait for a second at most
            if not ok then
                ngx.say("sub thread: failed to wait on sema: ", err)
            else
                ngx.say("sub thread: waited successfully.")
            end
        end

        local co = ngx.thread.spawn(handler)

        ngx.say("main thread: sleeping for a little while...")

        ngx.sleep(0.1)  -- wait a bit

        ngx.say("main thread: posting to sema...")

        sema:post(1)

        ngx.say("main thread: end.")
    }
}

```
The example location above produces a response output like this:
```
sub thread: waiting on sema...
main thread: sleeping for a little while...
main thread: posting to sema...
main thread: end.
sub thread: waited successfully.
```

2、在不相同的 `context` 线程中同步，`ngx.timer.at` 生成的线程与主线程不共享 `ngx.ctx`。

```
location = /t {
    content_by_lua_block {
        local semaphore = require "ngx.semaphore"
        local sema = semaphore.new()

        local outputs = {}
        local i = 1

        local function out(s)
            outputs[i] = s
            i = i + 1
        end

        local function handler()
            out("timer thread: sleeping for a little while...")

            ngx.sleep(0.1)  -- wait a bit

            out("timer thread: posting on sema...")

            sema:post(1)
        end

        assert(ngx.timer.at(0, handler))

        out("main thread: waiting on sema...")

        local ok, err = sema:wait(1)  -- wait for a second at most
        if not ok then
            out("main thread: failed to wait on sema: ", err)
        else
            out("main thread: waited successfully.")
        end

        out("main thread: end.")

        ngx.say(table.concat(outputs, "\n"))
    }
}
```
The example location above produces a response body like this.

```
main thread: waiting on sema...
timer thread: sleeping for a little while...
timer thread: posting on sema...
main thread: waited successfully.
main thread: end.
```

The same applies to different request contexts as long as these requests are served by the same nginx worker process.

[官网示例](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/semaphore.md)


### ngx.thread.spawn

生成一个新的用户级 "light thread" 执行函数，返回 "Lua coroutine" 对象。
在 `ngx.thread.spawn` return之前，函数会一直执行，直到函数 return、aborts with an error、yield。

在 "light thread" 触发异常终止，并不会影响到 "entry thread"，在当前的 "light thread" 下还可以执行此方法创建新的线程。

**注意事项**

1、"entry thread" 和 "light thread" 都执行完毕，请求才会终止。

```
location = /t {
    content_by_lua_block {

        local function handler()
            ngx.say("sub thread: start")
            while true do
              ngx.say("sub thread: loop")
              ngx.log(ngx.ERR,"sub thread: loop")
              ngx.sleep(0.2)
            end;
        end

        local co = ngx.thread.spawn(handler)
        ngx.say("main thread: sleeping for a little while...")
        ngx.sleep(1)  -- wait a bit
        ngx.say("main thread: end.")
    }
}
```

请求上面路径时，请求永远不会终止（正常没有创建子线程情况下，主线程会正常结束请求），即使 CTRL+C 强制终止请求，"sub thread" 依然会在后台执行。

2、"entry thread" 和" light thread" 在任意一个中，执行下面 ngx.exit, ngx.exec, ngx.redirect, or ngx.req.set_uri(uri, true) 
函数，请求会正常终止。

```
location = /t {
    content_by_lua_block {

        local function handler()
            ngx.say("sub thread: start")
            while true do
              ngx.say("sub thread: loop")
              ngx.log(ngx.ERR,"sub thread: loop")
              ngx.sleep(0.2)
            end;
        end

        local co = ngx.thread.spawn(handler)
        ngx.say("main thread: sleeping for a little while...")
        ngx.sleep(1)  -- wait a bit
        ngx.say("main thread: end.")
        ngx.exit(ngx.HTTP_OK)
    }
}
```

上面代码，请求会正常终止，"sub thread" 也会停止。

3、在 "entry thread" 中抛出错误，请求会终止、"sub thread" 也会停止。

```
location = /t {
    content_by_lua_block {

        local function handler()
            ngx.say("sub thread: start")
            while true do
              ngx.say("sub thread: loop")
              ngx.log(ngx.ERR,"sub thread: loop")
              ngx.sleep(0.2)
            end;
        end

        local co = ngx.thread.spawn(handler)
        ngx.say("main thread: sleeping for a little while...")
        ngx.sleep(1)  -- wait a bit
        ngx.say("main thread: end.")
        error
    }
}
```

4、"light thread" 可能会变成 "zombie" 状态，如果 "light thread" 已经终止（成功或失败），parent coroutine 依然存活，
并且 parent coroutine 没有通过 `ngx.thread.wait` 等待子线程终止。


5、函数 `ngx.thread.spwan` 特别适合并发流式请求，例如：

```
 -- query mysql, memcached, and a remote http service at the same time,
 -- output the results in the order that they
 -- actually return the results.

 local mysql = require "resty.mysql"
 local memcached = require "resty.memcached"

 local function query_mysql()
     local db = mysql:new()
     db:connect{
                 host = "127.0.0.1",
                 port = 3306,
                 database = "test",
                 user = "monty",
                 password = "mypass"
               }
     local res, err, errno, sqlstate =
             db:query("select * from cats order by id asc")
     db:set_keepalive(0, 100)
     ngx.say("mysql done: ", cjson.encode(res))
 end

 local function query_memcached()
     local memc = memcached:new()
     memc:connect("127.0.0.1", 11211)
     local res, err = memc:get("some_key")
     ngx.say("memcached done: ", res)
 end

 local function query_http()
     local res = ngx.location.capture("/my-http-proxy")
     ngx.say("http done: ", res.body)
 end

 ngx.thread.spawn(query_mysql)      -- create thread 1
 ngx.thread.spawn(query_memcached)  -- create thread 2
 ngx.thread.spawn(query_http)       -- create thread 3
```

[官网示例](https://github.com/openresty/lua-nginx-module#ngxthreadspawn)


### ngx.thread.wait

等待子线程执行结果，只有 "parent coroutine" 可以等待 "light thread"，其他情况会导致错误。

```
function f()
    ngx.sleep(0.2)
    ngx.say("f: hello")
    return "f done"
end

function g()
    ngx.sleep(0.1)
    ngx.say("g: hello")
    return "g done"
end

local tf, err = ngx.thread.spawn(f)
if not tf then
    ngx.say("failed to spawn thread f: ", err)
    return
end

ngx.say("f thread created: ", coroutine.status(tf))

local tg, err = ngx.thread.spawn(g)
if not tg then
    ngx.say("failed to spawn thread g: ", err)
    return
end

ngx.say("g thread created: ", coroutine.status(tg))

ok, res = ngx.thread.wait(tf, tg)
if not ok then
    ngx.say("failed to wait: ", res)
    return
end

ngx.say("res: ", res)

-- stop the "world", aborting other running threads
ngx.exit(ngx.OK)
```

执行结果

```
f thread created: running
g thread created: running
g: hello
res: g done
```

[官网示例](https://github.com/openresty/lua-nginx-module#ngxthreadwait)


### ngx.thread.kill

杀死子线程，也只有 "parent coroutine" 可以杀死子线程，由于内核的限制，运行的 "light thread"
正在 pending Nginx subrequests 是不可以被杀死的（如 initiated by ngx.location.capture for example)。


## Resty

### lua-resty-core

`Openresty` 内置核心库。

支持的方法：

* resty.core.hash
* resty.core.base64
* resty.core.uri
* resty.core.regex
* resty.core.exit
* resty.core.shdict
* resty.core.var
* resty.core.ctx
* get_ctx_table
* resty.core.request
* resty.core.response
* resty.core.misc
* resty.core.time
* resty.core.worker
* resty.core.phase
* resty.core.ndk
* resty.core.socket
* resty.core.param
* ngx.semaphore
* ngx.balancer
* ngx.ssl
* ngx.ssl.clienthello
* ngx.ssl.session
* ngx.re
* ngx.resp
* ngx.pipe
* ngx.process
* ngx.errlog
* ngx.base64

[官网示例](https://github.com/openresty/lua-resty-core)

### lua-resty-string

包含一些 md5、sha1 的处理函数。

支持的方法：

* resty.sha1
* resty.md5
* resty.sha224
* resty.sha256
* resty.sha512
* resty.sha384
* resty.random
* resty.aes

[官网示例](https://github.com/openresty/lua-resty-string)


## Test::Nginx

### Test

安装基础环境

```
$ cpanm Test::Base
$ cpanm Test::Nginx::Socket::Lua
```

验证
```
$ perl -MTest::Nginx::Socket::Lua -e 'print "OK\n"'
OK
```

测试示例

```
=== TEST 1: hello, world

--- config
location = /t {
    echo "hello, world!";
}

--- request
GET /t

--- response_body
hello, world!

# 确认响应状态码
--- error_code: 200

# 仅运行此block
--- ONLY

# 跳过此block
--- SKIP
```

执行

```
$ TEST_NGINX_BINARY="/usr/local/openresty/bin/openresty" prove -v
```


## 相关链接

[awesome-resty](https://github.com/bungle/awesome-resty)

[opm](https://github.com/openresty/opm)

[lua-nginx-module](https://github.com/openresty/lua-nginx-module)

[nginx embedded variables](http://nginx.org/en/docs/http/ngx_http_core_module.html#variables)

[HTTP status code](https://github.com/openresty/lua-nginx-module/#http-status-constants)

[HTTP common headers](https://itbilu.com/other/relate/EJ3fKUwUx.html)

[TCP keepalive](https://tldp.org/HOWTO/TCP-Keepalive-HOWTO/overview.html)

[Test::Nginx](https://openresty.gitbooks.io/programming-openresty/content/testing/test-nginx.html)

[LuaLint](http://lua-users.org/wiki/LuaLint)

[Lua CFunction、Valgrind](https://time.geekbang.org/column/article/100564?code=L6RL-eocu27wznXuQuV7XXvNA01tPBYxsdUgLU6wRLI%3D&screen=full)

[Lua CFunction](https://www.lua.org/pil/24.html)

[LuaJIT2](https://github.com/openresty/luajit2)

[Lua 5.1 Reference Manual](https://www.lua.org/manual/5.1/manual.html)

[Nginx API for Lua](https://github.com/openresty/lua-nginx-module/#nginx-api-for-lua)
