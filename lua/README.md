# Lua 教程

Lua 是一种强大、高效、轻量级、可嵌入的脚本语言。它支持过程式编程、面向对象编程、函数式编程、数据驱动编程和数据描述。

Lua 将简单的过程语法与基于关联数组和可扩展语义的强大数据描述结构相结合。Lua 是动态类型的，通过使用基于寄存器的虚拟机解释字节码来运行，并具有自动内存管理和增量垃圾收集，使其成为配置、脚本和快速原型设计的理想选择。

[官网](http://www.lua.org/about.html)

## 目录

一、[基本语法](#一基本语法)

二、[变量](#二变量)

三、[字符串](#三字符串)

四、[数值](#四数值)

五、[表](#五表)

六、[运算符](#六运算符)

七、[控制语句](#七控制语句)

八、[函数](#八函数)

九、[编译、执行和错误](#九编译执行和错误)

十、[迭代器与泛型 for](#十迭代器与泛型-for)

十一、[模块](#十一模块)


## 一、基本语法

写一个最最简单的程序，打印 Hello World。

```lua
print('hello world')
```

假定你把上面这句保存在 hello.lua 文件中，你在命令行只需要：

```shell
$ lua hello.lua 
```

### 1、块语句

Chunk 是一系列语句，Lua 执行的每一块语句，比如一个文件或者交互模式下的每一行都是一个 Chunk。
每个语句结尾的分号 `;` 是可选的，但如果同一行有多个语句最好用 `;` 分开。

```lua
a = 1 b = a * 2  -- ugly, but valid
```

### 2、注释

单行注释，两个减号是单行注释:

```lua
--  注释
```

多行注释：

```lua
--[[ 
print(10) -- no action (comment) 
--]]
```

### 3、标示符

Lua 标示符用于定义一个变量。以字母或者下划线开头，可以包含字母、下划线、数字序列。

最好不要使用下划线加大写字母的标示符，因为Lua的保留字也是这样的。

Lua 不允许使用特殊字符如 @, $, 和 % 来定义标示符。 

Lua 是一个区分大小写的编程语言。

以下字符为 Lua 的关键字，不能当作标识符。

```
and     break       do      else        elseif
end     false       for     function    if
in      local       nil     not         or
repeat  return      then    true        until 
while
```


## 二、变量

变量在使用前，需要在代码中进行声明，即创建该变量。

Lua 变量有三种类型：全局变量、局部变量、表中的域。


### 1、局部变量

使用 local 创建一个局部变量，与全局变量不同，局部变量只在被声明的那个代码块内有效。

代码块：指一个控制结构内，一个函数体。

```lua
x = 5

local i = 1

while i <= x do
    local a = 1
    local x = 10
    i = i + 1
end

print(i)  --> 6
print(x)  --> 5
print(a)  --> nil
```

应该尽可能的使用局部变量，有两个好处：

`1. 避免命名冲突；`

`2. 访问局部变量的速度比全局变量更快。`


### 2、全局变量

Lua 中的变量全是全局变量，哪怕是语句块或是函数里，除非用 local 显式声明为局部变量。

```lua
-- test.lua 文件脚本
a = 5               -- 全局变量
local b = 5         -- 局部变量

function joke()
    c = 5           -- 全局变量
    local d = 6     -- 局部变量
end

joke()
print(c,d)          --> 5 nil

do
    local a = 6     -- 局部变量
    b = 6           -- 对局部变量重新赋值
    print(a,b);     --> 6 6
end

print(a,b)      --> 5 6
```

全局变量不需要声明，给一个变量赋值后即创建了这个全局变量，访问一个没有初始化的全局变量也不会出错，只不过得到的结果是：nil.

```lua
print(b) --> nil 
b = 10 
print(b) --> 10
```

如果你想删除一个全局变量，只需要将变量赋值为 nil。

```lua
b = nil
print(b) --> nil
```

这样变量 b 就好像从没被使用过一样。换句话说, 当且仅当一个变量不等于 nil 时，这个变量存在。

### 3、类型和值

Lua 语言中有 8 种基本类型，nil(空)、boolean(布尔)、number(数值)、string(字符串)、userdata(用户数据)、
function(函数)、thread(线程)和table(表)。


## 三、字符串

### 1、赋值

字符串或串(string)是由数字、字母、下划线组成的一串字符。

Lua 语言中字符串可以使用以下三种方式来表示：

* 单引号间的一串字符。
* 双引号间的一串字符。
* [[ ]] 间的一串字符。

Lua 中字符串是不可以修改的，你可以创建一个新的变量存放你要的字符串，如下：

```lua
a = "a line"
b = 'another line'
```

为了风格统一，单引号和双引号最好使用一种，除非两种引号嵌套情况。

也可以使用 [[...]] 表示字符串，可以嵌套且不会解释转义序列，如果第一个字符是换行符会被自动忽略掉。
这种形式的字符串用来包含一段代码是非常方便的。

```lua
page = [[ 
<HTML> 
<HEAD> 
<TITLE>An HTML Page</TITLE> 
</HEAD> 
<BODY> 
Lua 
</BODY> 
</HTML> 
]]

io.write(page)
```

### 2、强制类型转换

Lua 会自动在 string 和 number 之间自动进行类型转换，当一个字符串使用算术操作符时，
string 就会被转成数字。

```lua
print("10" + 1)    --> 11.0
print("hello" + 1) -- ERROR (cannot convert "hello")
```

`..` 在 Lua 中是字符串连接符，当在一个数字后面写 `..` 时，必须加上空格以防止被解释错。

```lua
print("hello" .. " world")  --> hello world
```

可以调用 tostring() 将数字转成字符串，这种转换一直有效：

```lua
print(tostring(10) == "10")     --> true
print(tostring(nil) == "nil")   --> true
print(tostring("nil") == "nil") --> true
print(tostring({1, 3}))         --> table: 0x056e28d0
```

### 3、字符串标准库

Lua 提供了很多的方法来支持字符串的操作：

**1、字符串全部转为大写字母 `string.upper(s)`**。

```lua
s = string.upper("hello world")
print(s)        --> HELLO WORLD
```

**2、字符串全部转为小写字母 `string.lower(s)`**。

```lua
s = string.lower("HELLO WORLD")
print(s)        --> hello world
```

**3、字符串反转 `string.reverse(s)`**。

```lua
s = string.reverse('lua')
print(s)        --> aul
```

**4、字符串格式化  `string.format(formatstring, ...)`**。

格式化字符串中的指示符与 C 语言中函数 printf 的规则类似，一个指示符由一个百分号和一个代表格式化方式的字母组成:

```lua
%d      十进制整数
%x      十六进制整数
%f      浮点数
%s      字符串
```

示例

```lua
s = string.format("the value is:%d", 4)
print(s)        --> the value is:4
```

**5、字符串拷贝 `string.rep(s, n, sep)`**。

```lua
s = string.rep("abcd",2)
print(s)        --> abcdabcd
```

**6、字符串匹配 `string.match(s, pattern, init)`**。

string.match() 只寻找源字串 str 中的第一个配对. 参数 init 可选, 指定搜寻过程的起点, 默认为1。
在成功配对时, 函数将返回配对表达式中的所有捕获结果; 如果没有设置捕获标记, 则返回整个配对字符串。
当没有成功的配对时, 返回nil。

```lua
s = string.match("I have 2 questions for you.", "%d+ %a+")
print(s)        --> 2 questions
```

Looks for the first match of pattern in the string s. If it finds one, then match returns the captures from the pattern; otherwise it returns nil. If pattern specifies no captures, then the whole match is returned. A third, optional numerical argument init specifies where to start the search; its default value is 1 and can be negative.

**7、字符串匹配 `string.gmatch(s, pattern)`**。
返回一个迭代器函数，每一次调用这个函数，返回一个在字符串 str 找到的下一个符合 pattern 描述的子串。
如果参数 pattern 描述的字符串没有找到，迭代函数返回nil。

```lua
for word in string.gmatch("Hello Lua User", "%a+") do
    print(word)
end

--Hello
--Lua
--User
```
Returns an iterator function that, each time it is called, returns the next captures from pattern over string s. If pattern specifies no captures, then the whole match is produced in each call.

For this function, a '^' at the start of a pattern does not work as an anchor, as this would prevent the iteration.

**8、字符串截取 `string.sub(s, i, j)`**。

* s：要截取的字符串。
* i：截取开始位置。
* j：截取结束位置，默认为 -1，最后一个字符。

```lua
local s = string.sub('hello,world!', 4, 8)
print(s)        --> lo,wo
```

Returns the substring of s that starts at i and continues until j; i and j can be negative. If j is absent, then it is assumed to be equal to -1 (which is the same as the string length). In particular, the call string.sub(s,1,j) returns a prefix of s with length j, and string.sub(s, -i) returns a suffix of s with length i.

**9、在字符串中替换 `string.gsub(s, pattern, repl, n)`**。

s 为要操作的字符串， pattern 为被替换的字符，repl 要替换的字符，
n 替换次数（可以忽略，则全部替换），如：

```lua
s, n = string.gsub("aaaa", "a", "z", 3)
print(s, n)    --> zzza	3
```

Returns a copy of s in which all (or the first n, if given) occurrences of the pattern have been replaced by a replacement string specified by repl, which can be a string, a table, or a function. gsub also returns, as its second value, the total number of matches that occurred.

If repl is a string, then its value is used for replacement. The character % works as an escape character: any sequence in repl of the form %n, with n between 1 and 9, stands for the value of the n-th captured substring (see below). The sequence %0 stands for the whole match. The sequence %% stands for a single %.

If repl is a table, then the table is queried for every match, using the first capture as the key; if the pattern specifies no captures, then the whole match is used as the key.

If repl is a function, then this function is called every time a match occurs, with all captured substrings passed as arguments, in order; if the pattern specifies no captures, then the whole match is passed as a sole argument.

If the value returned by the table query or by the function call is a string or a number, then it is used as the replacement string; otherwise, if it is false or nil, then there is no replacement (that is, the original match is kept in the string).

Here are some examples:

```lua
x = string.gsub("hello world", "(%w+)", "%1 %1")
--> x="hello hello world world"

x = string.gsub("hello world", "%w+", "%0 %0", 1)
--> x="hello hello world"

x = string.gsub("hello world from Lua", "(%w+)%s*(%w+)", "%2 %1")
--> x="world hello Lua from"

x = string.gsub("home = $HOME, user = $USER", "%$(%w+)", os.getenv)
--> x="home = /home/roberto, user = roberto"

x = string.gsub("4+5 = $return 4+5$", "%$(.-)%$", function (s)
   return loadstring(s)()
 end)
--> x="4+5 = 9"

local t = {name="lua", version="5.1"}
x = string.gsub("$name-$version.tar.gz", "%$(%w+)", t)
--> x="lua-5.1.tar.gz"
```

**10、查找字符串 `string.find(s, pattern, init, plain)`**。

在一个指定的目标字符串 str 中搜索指定的内容 substr，如果找到了一个匹配的子串，
就会返回这个子串的起始索引和结束索引，不存在则返回 nil。

```lua
s, e = string.find("Hello Lua user", "Lua", 1)
print(s, e)     --> 7 9
```

Looks for the first match of pattern in the string s. If it finds a match, then find returns the indices of s where this occurrence starts and ends; otherwise, it returns nil. A third, optional numerical argument init specifies where to start the search; its default value is 1 and can be negative. A value of true as a fourth, optional argument plain turns off the pattern matching facilities, so the function does a plain "find substring" operation, with no characters in pattern being considered "magic". Note that if plain is given, then init must be given as well.

If the pattern has captures, then in a successful match the captured values are also returned, after the two indices.

**11、字符串长度 `string.len(s)`**。

Receives a string and returns its length. The empty string "" has length 0. Embedded zeros are counted, so "a\000bc\000" has length 5.

**12、转换整数为 ASCII 对应的字符  `string.char(...)`**。

函数接收零个或多个整数作为参数，然后将每个整数转换对应的字符，最后返回由这些字符连接而成的字符串。

```lua
print(string.char(97))          -->a
print(string.char(97, 98))      -->ab
```

**13、转换字符为 ASCII 对应的整数  `string.byte(s, i, j)`**。

函数返回字符 s 索引 i 到 j 之间(含)的所有字符的数值表示，i 和 j 都是可选参数。

```lua
print(string.byte('abc'))         --> 97
print(string.byte('abc', 1, 2))   --> 97 98
```


## 四、数值

nubmer 类型表示实数，Lua 中没有整数。一般有个错误的看法 CPU 运算浮点数比整数慢。事实不是如此，
用实数代替整数不会有什么误差（除非数字大于 100,000,000,000,000）。Lua 的 number 可以处理任何长整数不用担心误差。
你也可以在编译 Lua 的时候使用长整型或者单精度浮点型代替 number，
在一些平台硬件不支持浮点数的情况下这个特性是非常有用的，具体的情况请参考 Lua 发布版所附的详细说明。和其他语言类似，
数字常量的小数部分和指数部分都是可选的，数字常量的例子：

```
4   0.4   4.57e-3   0.3e12  5e+20
```

### 1、转换

可以调用 tonumber() 将数字转成字符串

```lua
print(tonumber("10") == 10) --> true

if type(v) == number then
    tonumber(v)
end
```

当被转换的参数非字符串数字时，结果为 nil

```lua
print(tonumber('abc') == nil)   --> true
print(tonumber({1, 3}) == nil)  --> true
print(tonumber('nil') == nil)   --> true
print(tonumber(nil) == nil)     --> true
```

### 2、随机数

math.random(): 不带参数调用时，返回一个在0（包含）到1（不包含）之间的伪随机数。

```lua
print(math.random())  -- 输出：0.0012512588889884 (每次运行结果会不同)
```

math.random(m, n): 带两个参数调用时，返回一个在m和n之间的整数伪随机数。m和n都必须是整数。

```lua
print(math.random(1, 10))  -- 输出：10 (每次运行结果会不同，但都在1到10之间, 包含1、10)
```

注意：在使用math.random之前，通常需要先调用math.randomseed(os.time())来设置随机数种子，以确保每次运行程序时生成的随机数序列都不同。

```lua
math.randomseed(os.time())
print(math.random(1, 10))  -- 输出：每次运行结果都会不同
```

## 五、表

table 是 Lua 中唯一的数据结构，其他语言所提供的其他数据结构比如：arrays、records、lists、queues、sets 等，
Lua 都是通过 table 来实现，并且在 lua 中 table 很好的实现了这些数据结构。

在传统的 C 语言或者 Pascal 语言中我们经常使用 arrays 和 lists（record+pointer）来实现大部分的数据结构，
在 Lua 中不仅可以用 table 完成同样的功能，而且 table 的功能更加强大。通过使用 table 很多算法的实现都简化了，
比如你在 lua 中很少需要自己去实现一个搜索算法，因为 table 本身就提供了这样的功能。


### 1、数组

lua 中通过整数下标访问表中的元素即可简单的实现数组。并且数组不必事先指定大小，大小可以随需要动态的增长。

```lua
array = {"Lua", "Tutorial"}

for i= 0, 2 do
   print(array[i])
end

nil
Lua
Tutorial
```

在 Lua 中习惯上数组的下表从 1 开始，Lua 的标准库与此习惯保持一致。

除了一维数组，还可以生成多维数据组：

```lua
array = {}
for i = 1, 3 do
   array[i] = {}
      for j = 1, 3 do
         array[i][j] = i*j
      end
end
```

### 2、字典

可以使用 table 数据结构实现字典:

```lua
map = {a = '1234', false, c = 10, 0}

print(map['a'])
print(map.b)
print(map.c)
print(map[1])
print(map[2])

1234
nil
10
false
0
```

字典可以通过两种方式获取 value 值，字典的 key 不影响列表的下标。


### 3、泛型 for 迭代器

数组有多种循环打印方式，先看一下 pairs 与 iparis 的区别：


```lua
tb = {a = '1234', false, nil, b = nil, c = 10, 0}

for k, v in pairs(tb) do
    print(k, v)
end

1	false
3	0
c	10
a	1234
```

注意查看打印的顺序，以及下标。nil 的值会自动跳过。

再看一下 ipairs 的使用：

```lua
tb = {a = '1234', false, c = 10, 0}

for k, v in ipairs(tb) do
    print(k, v)
end

1	false
2	0


tb = {'1234', false, nil, 10, 0}

for k, v in ipairs(tb) do
    print(k, v)
end

1	1234
2	false
```

ipairs 获取数组的下标和值，不会循环 map 的值。并且遇到 nil 后，会自动停止循环。

### 4、长度

可以通过 `#` 号来获取 `table` 中元素的个数，但要注意不同的版本对相同的 `table` 操作，可能结果不同。

自动忽略 `map` 类型，只会统计从 1 开始的下标。

```lua
tb = {'1234', a = 1, b = 2}
print(#tb)

1
```

当出现 `nil` 时，也会统计在内。

```lua
tb = {'a', 'b', nil, 'c', 'd'}
print(#tb)
tb[10] = 'h'
print(#tb)

-- Lua 5.1.4
5
10
```

谨慎在 luajit 中使用 `#` 统计元素个数，例如下面的统计结果就让人很迷惑：

```lua
tb = {'key1', 100, '123', nil, 'key2'}
print(#tb)

tb = {'key1', 100, nil, '123','key2'}
print(#tb)

-- luajit-2.0.0
3
5
```

有些遇到 `nil` 会在索引中断的地方停止计数。可以使用以下方法来代替：

```lua
function table_len(t)
  local len=0
  for k, v in pairs(t) do
    len=len+1
  end
  return len;
end
```

### 5、table 操作

以下列出了 `table` 操作常用的方法：

① 元素拼接，table.concat(list, sep, i, j)：

列出参数中指定 `table` 的数组部分从 `i` 位置到 `j` 位置的所有元素, 元素间以指定的分隔符 `sep` 隔开。

```lua
str = table.concat({ 'a', 'b', 1, 'c', 'd' }, '+', 1, 4)
print(str)
```

② 元素插入，table.insert(list, pos, value):

在 `table` 的数组部分指定位置 `pos` 插入值为 `value` 的一个元素. pos 参数可选, 默认为数组部分末尾。

```lua
tb = {'a', 'b'}
table.insert(tb, 'c')
print(tb[3])

table.insert(tb, 1, 'i')
print(tb[1])

c
i
```

③ 元素删除，table.remove(list, pos):

返回 `table` 数组部分位于 `pos` 位置的元素。其后的元素会被前移. `pos` 参数可选, 默认为 `table` 长度, 即从最后一个元素删起。

```lua
tb = {'a', 'b'}

table.remove(tb)
print(tb[2])

nil
```

④ sort排序，table.sort(list, comp)：

`table` 根据自定义排序方法 `comp` 进行排序。


### 6、如何判断 table 为空？

通过上面的介绍，使用 '#' 来统计元素个数是否为 0 显示是不正确的。

```
local tb = { key1 = 'value1' }

if #tb == 0 then
   print('empty')
end

empty
```

可以使用next函数进行遍历，判断是否为空。

```
local tb = { key1 = 'value1' }

if next(tb) == nil then
    print('empty')
else
    print('not empty')
end

not empty
```

## 六、运算符

### 1、算术运算符

二元运算符：+ - * / ^ % (加减乘除幂取模) 。

一元运算符：- (负值) 。

这些运算符的操作数都是实数。

```lua
local n = 3 / 2
print(n)

-- 1.5
```

### 2、关系运算符

```
<   >   <=  >=  ==  ~=
```

这些操作符返回结果为 false 或者 true；== 和 ~= 比较两个值，如果两个值类型不同，
Lua 认为两者不同；nil 只和自己相等。Lua 通过引用比较 tables、userdata、functions。
也就是说当且仅当两者表示同一个对象时相等。

```lua
a = {}; a.x = 1; a.y = 0
b = {}; b.x = 1; b.y = 0
c = a
print(a == c)      -- true
print(b == c)      -- false
```
Lua 比较数字按传统的数字大小进行，比较字符串按字母的顺序进行，但是字母顺序依赖于本地环境。
当比较不同类型的值的时候要特别注意：

```lua
"0" == 0        -- false 
2 < 15          -- true 
"2" < "15"      -- false (alphabetical order!)
```

为了避免不一致的结果，混合比较数字和字符串，Lua 会报错，比如：2 < "15"。

### 3、逻辑运算符

```lua
and   or  not
```

逻辑运算符认为 false 和 nil 是假（false），其他为真，0 也是 true。
and 和 or 的运算结果不是 true 和 false，而是和它的两个操作数相关。

```lua
a and b     -- 如果 a 为 false，则返回 a，否则返回 b 
a or b      -- 如果 a 为 true，则返回 a，否则返回 b
```
例如：
```lua
print(4 and 5)      --> 5 
print(nil and 13)   --> nil 
print(false and 13) --> false 
print(4 or 5)       --> 4 
print(false or 5)   --> 5
```

一个很实用的技巧：如果 x 为 false 或者 nil 则给 x 赋初始值 v。

```lua
x = x or v
```

等价于

```lua
if not x then
    x = v 
end
```

and 的优先级比 or 高。

C 语言中的三元运算符：

```C
a ? b : c
```

在 Lua 中可以这样实现：

```lua
(a and b) or c
```

not 的结果一直返回 false 或者 true。

```lua
print(not nil)      --> true 
print(not false)    --> true 
print(not 0)        --> false 
print(not not nil)  --> false
```

### 4、优先级

从高到低的顺序：

```lua
^ 
not - (unary) 
* / 
+ - 
.. 
<   >   <=  >=  ~=  == 
and 
or
```

除了和外所有的二元运算符都是左连接的。

```lua
a+i < b/2+1         <-->    (a+i) < ((b/2)+1) 
5+x^2*8             <-->    5+((x^2)*8) 
a < y and y <= z    <-->    (a < y) and (y <= z) 
-x^2                <-->    -(x^2) 
x^y^z               <-->    x^(y^z)
```

### 5、位运算

位运算包括: & | ~ >> << 及 一元运算符 ~

移位操作都会用 0 填充空的位，这种被称为逻辑移位，Lua 语言没有提供算术移位（可以通过向下取整法，除以合适的 2 的整次幂实现算数右移，如x//16与算数右移 4 位等价）。
移位数是负数表示向相反的方向移位，即 a>>n 与 a<<-n 等价。

整型占用 64 位，取值范围 -2^63 ~ 2^63-1。
Lua 语言不显示支持无符号整型数，如果需要处理的话，可以采用其他的方式。

```lua
print(3<<62)      --> -4611686018427387904
```

问题不在于常量本身，而在于输出方式，可以使用选项 %u 或 %x 在函数 string.format 中指定以无符号整型数进行输出。

```lua
print(string.format("%u", 3<<62))      --> 13835058055282163712
print(string.format("0x%x", 3<<62))    --> 0xc000000000000000
```

无符号整型数的加减与有符号整型数操作一致。无符号数的比较和除法用到的较少，就不写了。

```lua
print(string.format("%u", (3<<62)+1))       --> 13835058055282163713
```


## 七、控制语句

控制结构的条件表达式结果可以是任何值，**在控制结构的条件中除了 false 和 nil 为假，
其他值都为真。所以 Lua 认为 0 和空串都是真**。

### 1、if else

if 语句，有三种形式：

```lua
if conditions then
    then-part 
end; 
    
if conditions then
    then-part 
else 
    else-part 
end; 

if conditions then
    then-part 
elseif conditions then
    elseif-part 
.. --->多个 elseif 
else 
    else-part 
end;
```

### 2、while 

while 语句：

```lua
while condition do 
    statements; 
end;
```

### 3、for 

for 语句有两大类：

第一，数值 for 循环：

```lua
for var = exp1, exp2, exp3 do 
    loop-part 
end
```

for 将用 exp3 作为 step 从 exp1（初始值）到 exp2（终止值），执行 loop-part。其中
exp3 可以省略，默认 step=1。

1. 三个表达式只会被计算一次，并且是在循环开始前。
```lua
for i = 10, 1, -1 do 
    print(i)           -- 10，9 ... 1
end

for i = 1, 10, 1 do 
    print(i)           -- 1，2 ... 10
end
```

2. 控制变量 var 是局部变量自动被声明,并且只在循环内有效。

```lua
for i = 1, 10 do 
    print(i) 
end 
max = i                 -- probably wrong! 'i' here is global
```
如果需要保留控制变量的值，需要在循环中将其保存。

```lua
-- find a value in a list 
local found = nil
for i = 1, a.n do
    if a[i] == value then
        found = i -- save value of 'i' 
        break 
    end 
end 
print(found)
```

3. 循环过程中不要改变控制变量的值，那样做的结果是不可预知的。如果要退出循环，使用 `break` 语句。
   
第二，范型 for 循环：

前面已经见过一个例子：

```lua
-- print all values of array 'a' 
for i,v in ipairs(a) do print(v) end
```
范型 for 遍历迭代子函数返回的每一个值。
再看一个遍历表 key 的例子：

```lua
-- print all keys of table 't' 
for k in pairs(t) do print(k) end
```

范型 for 和数值 for 有两点相同：
1. 控制变量是局部变量
2. 不要修改控制变量的值

### 4、break 与 return 语句

break 语句用来退出当前循环（for,repeat,while）。在循环外部不可以使用。
return 用来从函数返回结果，当一个函数自然结束结尾会有一个默认的 return。（这种函数类似 pascal 的过程）

没有 continue， 没有 continue !!

### 5、作用域块

lua

```lua
a = 0

for i = 1, 2 do
    a = a + 1
    b = 1
end

print(i)    -- nil
print(a)    -- 2
print(b)    -- 1
```

python 与 lua 有所不同
```python
a = 1

for i in range(2):
    a += 1
    b = 1

print(i)    # 1
print(a)    # 3
print(b)    # 1
```

java 与 python 又完全不同

``` java
public class FirstSample
{
  public static void main(String[] args)
  {
    int a = 1;
    for (int i=0; i<=1; i++) {
      a++;
      int b = 1;
    }
    System.out.println(i);  # 错误: 找不到符号
    System.out.println(a);  # 3
    System.out.println(b);  # 错误: 找不到符号
  }
}
```

相同点：
* 代码块可以修改外部变量，并使外部变量生效

不同点：
* java 代码块中定义的变量，外部不可以访问，python 和 lua 可以
* for 语句定义的变量 i，python 可以访问 lua 不行


## 八、函数

函数有两种用途：
* 完成指定的任务，这种情况下函数作为调用语句使用；
* 计算并返回值，这种情况下函数作为赋值语句的表达式使用。

Lua 也提供了面向对象方式调用函数的语法，比如 o:foo(x) 与 o.foo(o, x) 是等价的，

Lua 函数实参和形参的匹配与赋值语句类似，多余部分被忽略，缺少部分用 nil 补足。

```lua
function f(a, b) 
    return a or b 
end

f(3) a=3, b=nil 
f(3, 4) a=3, b=4 
f(3, 4, 5) a=3, b=4 (5 is discarded)
```

### 1、 返回多个结果值

Lua 函数可以返回多个结果值，比如 string.find，其返回匹配串"开始和结束的下标"（如果不存在匹配串返回 nil）。

```lua
s, e = string.find("hello Lua users", "Lua") 
print(s, e)     --> 7 9
```

函数多值返回的特殊函数 unpack，接受一个数组作为输入参数，返回数组的所有元素。

```lua
t = {10, 20, 30}
print(unpack(t))    --> 102030
```

在 C 语言中可以使用函数指针调用可变的函数，可以声明参数可变的函数，但不能两者同时可变。
在 Lua 中如果你想调用可变参数的可变函数只需要这样。

```lua
f = string.find 
a = {"hello", "ll"} 
print(f(unpack(a))) --> 3 4
```

预定义的 `unpack` 函数是用 C 语言实现的，我们也可以用 Lua 来完成。

```lua
function unpack(t, i)
    i = i or 1
    if t[i] then
        return t[i], unpack(t, i + 1)
    end
end
```

### 2、 可变参数

Lua 函数可以接受可变数目的参数，和 C 语言类似在函数参数列表中使用三点（...）
表示函数有可变的参数。Lua 将函数的参数放在一个叫 arg 的表中，除了参数以外，
arg 表中还有一个域 n 表示参数的个数。

```lua
-- 测试函数内并没有 arg 变量！！
function g (a, b, ...) end

g(3)            a=3, b=nil, arg={n=0} 
g(3, 4)         a=3, b=4, arg={n=0} 
g(3, 4, 5, 8)   a=3, b=4, arg={5, 8; n=2}
```

如上面所示，Lua 会将前面的实参传给函数的固定参数，后面的实参放在 arg 表中。

如果我们只想要 string.find 返回的第二个值，一个典型的方法是使用虚变量（下划线）。

```lua
local _, x = string.find(s, p) 
-- now use `x' 
...
```

### 3、 命名参数

Lua 的函数参数是和位置相关的，调用时实参会按顺序依次传给形参。有时候用名字指定参数是很有用的，
比如 rename 函数用来给一个文件重命名，有时候我们我们记不清命名前后两个参数的顺序了：

```lua
function rename(old, new)
    print(old)
    print(new)
end

-- invalid code 
rename(old="old.lua", new="new.lua")
```

上面这段代码是无效的，Lua 可以通过将所有的参数放在一个表中，把表作为函数
的唯一参数来实现上面这段伪代码的功能。因为 Lua 语法支持函数调用时实参可以是表的构造。

```lua
rename({old="old.lua", new="new.lua"})
```

根据这个想法我们重定义了 rename：

```lua
function rename (arg) 
    return os.rename(arg.old, arg.new) 
end
```

当函数的参数很多的时候，这种函数参数的传递方式很方便的。

### 4、 闭包

当一个函数内部嵌套另一个函数定义时，内部的函数体可以访问外部的函数的局部变量，
这种特征我们称作词法定界。虽然这看起来很清楚，事实并非如此，
词法定界加上第一类函数在编程语言里是一个功能强大的概念，很少语言提供这种支持。

看下面的代码：

```lua
function newCounter()
    local i = 0
    return function()
        -- anonymous function 
        i = i + 1
        return i
    end
end

c1 = newCounter()
print(c1()) --> 1 
print(c1()) --> 2

c2 = newCounter()
print(c2()) --> 1 
```

匿名函数使用 upvalue(外部局部变量) i 保存他的计数，当我们调用匿名函数的时候 i 已经超出了作用范围，
因为创建 i 的函数 newCounter 已经返回了。然而 Lua 用闭包的思想正确处理了这种情况。
简单的说闭包是一个函数加上它可以正确访问的 upvalues。如果我们再次调用 newCounter，
将创建一个新的局部变量 i，因此我们得到了一个作用在新的变量 i 上的新闭包。

### 5、 非全局函数

Lua 中函数可以作为全局变量也可以作为局部变量，我们已经看到一些例子：
函数作为 table 的域（大部分 Lua 标准库使用这种机制来实现的比如 io.read、math.sin）。
这种情况下，必须注意函数和表语法：

1. 表和函数放在一起

```lua
Lib = {} 
Lib.foo = function (x,y) return x + y end
Lib.goo = function (x,y) return x - y end
```

2. 使用表构造函数

```lua
Lib = {
    foo = function (x,y) return x + y end,
    goo = function (x,y) return x - y end
} 
```

3. Lua 提供另一种语法方式

```lua
Lib = {}
function Lib.foo (x, y)
    return x + y
end
function Lib.goo (x, y)
    return x - y
end
```

当我们将函数保存在一个局部变量内时，我们得到一个局部函数，也就是说局部函
数像局部变量一样在一定范围内有效。这种定义在包中是非常有用的：因为 Lua 把 chunk
当作函数处理，在 chunk 内可以声明局部函数（仅仅在 chunk 内可见），词法定界保证了包内的其他函数可以调用此函数。
下面是声明局部函数的两种方式：

1. 方式一

```lua
local f = function(...)
    ...
end
local g = function(...)
    ...
    f() -- external local `f' is visible here 
    ...
end
```

2. 方式二

```lua
local function f (...) 
    ... 
end
```

在定义非直接递归局部函数时要先声明然后定义才可以：

```lua
local f, g -- `forward' declarations 
function g ()
    ...
    f()
    ...
end
function f ()
    ...
    g()
    ...
end
```

### 6、 正确的尾调用

Lua 中函数的另一个有趣的特征是可以正确的处理尾调用。
尾调用是一种类似在函数结尾的 goto 调用，当函数最后一个动作是调用另外一个函数时，
我们称这种调用尾调用。例如：

```lua
function f(x) 
    return g(x) 
end
```

g 的调用是尾调用。

例子中 f 调用 g 后不会再做任何事情，这种情况下当被调用函数 g 结束时程序不需要返回到调用者 f；
所以尾调用之后程序不需要在栈中保留关于调用者的任何信息。
一些编译器比如 Lua 解释器利用这种特性在处理尾调用时不使用额外的栈，我们称这种语言支持正确的尾调用。 

由于尾调用不需要使用栈空间，那么尾调用递归的层次可以无限制的。例如下面调用不论 n 为何值不会导致栈溢出。

```lua
function foo (n) 
    if n > 0 then return foo(n - 1) end
end
```

需要注意的是：必须明确什么是尾调用。
一些调用者函数调用其他函数后也没有做其他的事情但不属于尾调用。比如：

```lua
function f (x)  
    g(x) 
    return 
end
```

上面这个例子中 f 在调用 g 后，不得不丢弃 g 地返回值，所以不是尾调用，同样的
下面几个例子也不是尾调用：

```lua
return g(x) + 1     -- must do the addition 
return x or g(x)    -- must adjust to 1 result 
return (g(x))       -- must adjust to 1 result
```

Lua 中类似 return g(...)这种格式的调用是尾调用。但是 g 和 g 的参数都可以是复杂
表达式，因为 Lua 会在调用之前计算表达式的值。例如下面的调用是尾调用：

```lua
return x[i].foo(x[j] + a*b, i + j)
```


## 九、编译、执行和错误

### 1、编译

与函数 `dofile` 类似，函数 `loadfile` 也是从文件中加载 Lua 代码段，但它不会运行代码，只是编译代码，
然后将编译后的代码段作为函数返回。

两者的关系可以看成以下：

```lua
function dofile(filename)
    local f = assert(loadfile(filename))
    return f()
end 
```

当发生错误时，函数 `loadfile` 会返回 nil 及错误信息。

```lua
local f, err = loadfile('not_exist')
print(f)            --> nil
print(err)          --> cannot open not_exist: No such file or directory
```

函数 `load` 与函数 `loadfile` 类似，不同之处在于该函数从一个字符串或函数中读取代码段，而不是从文件中读取。

```lua
f = load("i = i + 1")
i = 0
f();
print(i)            --> 1
```

尽管函数 `load` 的功能很强大，但还是应该谨慎使用，该函数开销较大并且可能会引起诡异的问题。

与 `loadfile` 一样， 当发生错误时，函数 `load` 会返回 nil 及错误信息。

```lua
local f, err = load('error chunk')
print(f)            --> nil
print(err)          --> [string "error chunk"]:1: syntax error near 'chunk'
```

需要注意的是函数 `load` 总是在全局环境中编译代码段。

```lua
i = 32
local i = 0
f = load("i = i + 1; print(i)")
f()             --> 33
```

如果需要对表达式求值，可以在表达式前添加 return。

```lua
f = load("return 100")
print(f())             --> 100
```

也可以使用读取函数作为函数 `load` 的第一个参数，读取函数可以分几次返回一段程序，函数 `load` 会不断调用读取函数直到读取函数返回 nil（表示程序段结束）。

```lua
count = 0
local loader = function()
    count = count + 1
    if count == 1 then
        return 'local a = '
    elseif count == 5 then
        return '; print(a)'
    elseif count < 5 then
        return count
    end
    return nil
end

f = load(loader)
f()         --> 234  
```

函数 `loadfile` 加载的文件，文件内的变量，只有在运行代码才会定义它。

```lua
-- 文件 'foo.lua'
function foo(x)
    print(x)
end

f = load("foo.lua")
print(foo)          --> nil
f()                 --> 运行代码
foo("ok")           --> ok
```

### 2、预编译

执行下面命令可以生成预编译文件。

```shell
$ luac -o prog.lc prog.lua
```

### 3、错误处理和异常

函数 `pcall` 会以一种保护模式来调用它的第一个参数，以便捕获该函数执行中的错误。无论是否有错误发生，函数 `pcall` 都不会引发错误。
如果没有错误发生，那么 `pcall` 返回 true 及被调用函数的所有返回值，否则，则返回 false 及错误信息。

```lua
local status, err = pcall(function()
    error({ code = 121 })
end)
print(status)           --> false
print(err.code)         --> 121
```

### 4、错误信息和栈回溯

函数 error 还有第二个可选参数 level，用于指出向函数调用层次中的哪层函数报告错误。

函数 `xpcall` 与 `pcall` 类似，第二个参数是消息出来函数，可以获取发出错误时，完整的调用栈的栈回溯，如下。

```shell
local function foo(str)
    if type(str) ~= 'string' then
        error('string expected',2)
    end
end


local function test()
    foo(123)
end

local status, err = xpcall(test, debug.traceback)
print(status)           --> false
print(err)
```

[《using-pcall》](https://riptutorial.com/lua/example/16000/using-pcall)


## 十、迭代器与泛型 for

### 1、迭代器与闭包

迭代器需要保留上一次成功调用的状态和下一次成功调用的状态，也就是他知道来自于哪里和将要前往哪里。
闭包提供的机制可以很容易实现这个任务。

举一个简单的例子，我们为一个 list 写一个简单的迭代器，
与 ipairs()不同的是我们实现的这个迭代器返回元素的值而不是索引下标：

```lua
function list_iter(t)
    local i = 0
    local n = table.getn(t)
    return function()
        i = i + 1
        if i <= n then
            return t[i]
        end
    end
end
```

可以在 while 语句中使用这个迭代器：

```lua
t = { 10, 20, 30 }
iter = list_iter(t) -- creates the iterator 
while true do
    local element = iter() -- calls the iterator 
    if element == nil then
        break
    end
    print(element)
end 
```

我们设计的这个迭代器也很容易用于范性 for 语句

```lua
t = {10, 20, 30} 
for element in list_iter(t) do  
    print(element) 
end
```

当迭代器返回 nil 时循环结束, 仅当第一个值为 nil 时迭代停止。

```lua
local function list_iter(t)
    local i = 0
    local n = table.getn(t)
    return function()
        i = i + 1
        if i == 3 then
            return i, nil
        elseif i == 4 then
            return nil, t[i]
        end

        if i <= n then
            return i, t[i]
        end
    end
end

local t = {10, 20, 30, 40, 50}
for i,element in list_iter(t) do   
    print(i, ':', element)
end
--1:10
--2:20
--3:
```

### 2、 范性 for 的语义

范性 for 在自己内部保存三个值:迭代函数，状态常量和控制变量。

```lua
for <var-list> in <exp-list> do
    <body> 
end
```

我们称变量列表中第一个变量为控制变量，其值为 nil 时循环结束。

范性 for 的执行过程：

* 首先，初始化，计算 in 后面表达式的值，表达式应该返回范性 for 需要的三个值：迭代函数，状态常量和控制变量；
  与多值赋值一样，如果表达式返回的结果个数不足三个会自动用 nil 补足，多出部分会被忽略。
* 第二，将状态常量和控制变量作为参数调用迭代函数（注意：对于 for 结构来说，
  状态常量没有用处，仅仅在初始化时获取他的值并传递给迭代函数）。
* 第三，将迭代函数返回的值赋给变量列表。
* 第四，如果返回的第一个值为 nil 循环结束，否则执行循环体。
* 第五，回到第二步再次调用迭代函数。

更精确的来说：

```lua
for var_1, ..., var_n in explist do block end
```

等价于

```lua
do 
local _f, _s, _var = explist 
while true do
    local var_1, ... , var_n = _f(_s, _var) 
        _var = var_1 
    if _var == nil then break end
        block 
end 
end
```

### 3、 无状态的迭代器

利用状态常量和控制变量的值作为参数被调用。

```lua
function iter(a, i)
    i = i + 1
    local v = a[i]
    if v then
        return i, v
    end
end 

function ipairs(a) 
    return iter, a, 0
end

a = {"one", "two", "three"} 
for i, v in ipairs(a) do 
    print(i, v) 
end
```


## 十一、模块

模块类似于一个封装库，从 Lua 5.1 开始，Lua 加入了标准的模块管理机制，可以把一些公用的代码放在一个文件里，
以 API 接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度。

Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，
然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。以下为创建自定义模块 module.lua，文件代码格式如下：

```lua
_M = {}

local mt = { __index = _M }

function _M.func1()
    io.write("这是一个公有函数！\n")
end

local function func2()
    print("这是一个私有函数！")
end

function _M.func3()
    func2()
end

function _M.new()
    return setmetatable({
        constant = "这是一个常量",
    }, mt)
end

return _M
```

由上可知，模块的结构就是一个 table 的结构，因此可以像操作调用 table 里的元素那样来操作调用模块里的常量或函数。

上面的 func2 声明为程序块的局部变量，即表示一个私有函数，因此是不能从外部访问模块里的这个私有函数，必须通过模块里的公有函数来调用。

```lua
local module = require "module"

local md = module.new()

print(md.constant)

md.func1()

md.func3()
```

以上代码执行结果为：

```lua
这是一个常量
这是一个公有函数！
这是一个私有函数！
```

### 加载机制

对于自定义的模块，模块文件不是放在哪个文件目录都行，函数 require 有它自己的文件路径加载策略，它会尝试从 Lua 文件或 C 程序库中加载模块。

require 用于搜索 Lua 文件的路径是存放在全局变量 package.path 中，当 Lua 启动后，会以环境变量 LUA_PATH 的值来初始这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。

```shell
$ lua
Lua 5.3.0  Copyright (C) 1994-2015 Lua.org, PUC-Rio
>  print(package.path)
/usr/local/share/lua/5.3/?.lua;/usr/local/share/lua/5.3/?/init.lua;/usr/local/lib/lua/5.3/?.lua;/usr/local/lib/lua/5.3/?/init.lua;./?.lua;./?/init.lua
```

在当前用户根目录下打开 .profile 文件（没有则创建，打开 .bashrc 文件也可以），例如把 "~/lua/" 路径加入 LUA_PATH 环境变量里：

```shell
#LUA_PATH
export LUA_PATH="~/lua/?.lua;;"
```

文件路径以 ";" 号分隔，最后的 2 个 ";;" 表示新加的路径后面加上原来的默认路径。

接着，更新环境变量参数，使之立即生效。

```shell
source ~/.profile
```


## 相关链接

[Lua 5.1 Reference Manual](https://www.lua.org/manual/5.1/manual.html)
