# LuaJIT FFI

The FFI library allows calling external C functions and using C data structures from pure Lua code.

The FFI library largely obviates the need to write tedious manual Lua/C bindings in C. 
No need to learn a separate binding language — it parses plain C declarations! 
These can be cut-n-pasted from C header files or reference manuals. 
It's up to the task of binding large libraries without the need for dealing with fragile binding generators.

## 一、使用

自定义的 C 函数:

```C
$ cat t.c 
int add(int x, int y)
{
    return x + y;
}
```

将函数编译成链接库：

```shell
$ gcc -g -o libt.so -fpic -shared t.c
```

添加路径到全局变量：

```shell
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:your_lib_path
```

调用自己的函数，可以直接使用 ffi.load 返回的变量调用，下面我们看一个简单的例子：

```lua
local ffi = require('ffi')
t = ffi.load('t')

ffi.cdef[[
    int add(int x, int y);
]]

ret = t.add(1, 20)
print(ret)

-- 21
```

## 二、类型转换

### 1、字符串

可以使用 Lua string 对象来初始化字符串 cdata 对象。

C ：
```C
#include <stdio.h>

void print(const char *s)
{
    printf("%s\n", s);
}
```

Lua :
```Lua
local ffi = require('ffi')
local t = ffi.load("t")

ffi.cdef[[
    void print(const char *s);
]]

-- right
a = ffi.new("const char*", "Hello World")
b = ffi.new("const char[1]", "Hello World")
f = ffi.new("char[11]", "Hello World")

-- wrong
-- c = ffi.new("const char", "Hello World") -- luajit: test.lua:14: cannot convert 'string' to 'const char'
-- d = ffi.new("char","Hello World")        -- luajit: test.lua:15: cannot convert 'string' to 'char'
-- e = ffi.new("char*", "hello world")      -- luajit: test.lua:16: cannot convert 'string' to 'char *'

t.print(a)
t.print(b)
t.print(f)

--Hello World
--H
--Hello World
```


### 2、数值

可以使用 Lua number 对象来初始化 int cdata 对象。

C ：
```C
#include <stdio.h>

int add(int x, int y)
{
    return x+y;
}

int addp(int *x, int *y)
{
    return *x+*y;
}
```
Lua :

```Lua
local ffi = require('ffi')
local t = ffi.load("t", true)

ffi.cdef[[
    int add(int x, int y);
    int addp(int *x, int *y);
]]

a = ffi.new("int", 10)
b = ffi.new("int", 10)
print(t.add(a, b))

a = ffi.new("int[1]", {10})
b = ffi.new("int[1]", {10})
--b = ffi.new("int*", 10)    luajit: test.lua:15: cannot convert 'number' to 'int *'
--b = ffi.new("int*", {10})  luajit: test.lua:16: cannot convert 'table' to 'int *'
print(t.addp(a, b))

-- wrong
--a = ffi.new("int[1]", {10})
--b = ffi.new("int", 10)
--print(t.addp(a, b))        luajit: test.lua:16: bad argument #2 to 'addp' (cannot convert 'int' to 'int *')
```

### 3、结构 struct

首先是一个 C 程序，我们使用构造的 cadata 对象来调用该函数：

C ：

```C
#include <stdio.h>

struct constr_t {
    int a;
    int b;
    struct innerstr {
        int x;
        int y;
    } c;
};

void print_constr_t(struct constr_t t)
{
    printf("a:%d\n", t.a);
    printf("b:%d\n", t.b);
    printf("c.x:%d\n", t.c.x);
    printf("c.y:%d\n", t.c.y);
}

void print_pconstr_t(struct constr_t *t)
{
    printf("a:%d\n", t->a);
    printf("b:%d\n", t->b);
    printf("c.x:%d\n", t->c.x);
    printf("c.y:%d\n", t->c.y);
}
```

Lua :

```lua
local ffi = require('ffi')
local t = ffi.load("t", true)

ffi.cdef[[
    struct constr_t {
        int a;
        int b;
        struct innerstr {
            int x;
            int y;
        } c;
    };
    void print_constr_t(struct constr_t t);
    void print_pconstr_t(struct constr_t *t);
]]

a = ffi.new("struct constr_t", {1, 2, {10, 11}})
t.print_constr_t(a)
t.print_pconstr_t(a)

--a:1
--b:2
--c.x:10
--c.y:11

b = ffi.new("struct constr_t[1]", {{1, 2, {10, 11}}})
t.print_pconstr_t(b)

--a:1
--b:2
--c.x:10
--c.y:11

-- wrong
-- c = ffi.new("struct constr_t*", {1, 2, {10, 11}}) luajit: test.lua:24: cannot convert 'table' to 'struct constr_t *'

```



[《LuaJIT 官网》](https://luajit.org/ext_ffi.html)