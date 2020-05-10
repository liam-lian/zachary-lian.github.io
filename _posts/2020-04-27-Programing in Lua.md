---
layout: post
category: "Lua"
title:  "Programing in Lua
---



## 起点

#### Chunks  

指的是一个代码段，也就是一个程序块。在交互模式下，每一条语句是一个chunks

先运行a再运行b

```bash
lua -la -lb
```

`dofile` 函数加载文件并执行它，在调试的时候实用  

```lua
dofile("lib1.lua")
```

命令行参数保存在名为arg的表中

```bash
lua xx.lua s1
-- xx.lua中arg表就是{"s1"}
```



## 类型和值

```lua
n=tonumber("10.01")    --字符串转数字
tostring(10) == "10"
```

## 表达式

#### 关系运算

Lua 通过引用比较 tables、 userdata、 functions，也就是说当且仅当两者表示同一个对象时相等。  

#### 逻辑运算

false 和 nil 是假（false），其他为真， 0 也是 true.
**and 和 or 的运算结果不是 true 和 false，而是和它的两个操作数相关。**

a and b -->如果 a 为 false，则返回 a，否则返回 b

a or b -->如果 a 为 true，则返回 a，否则返回 b

```lua
print(4 and 5) --> 5
print(nil and 13) --> nil
print(4 or 5) --> 4
print(false or 5) --> 5
-- 一个很实用的技巧：如果 x 为 false 或者 nil 则给 x 赋初始值 v
x = x or v
```

C 语言中的三元运算符

```lua
a ? b : c
```

在 Lua 中可以这样实现：

```lua
(a and b) or c  
```

#### 表的构造

从输入中读取，构造反向链表

```lua
list = nil
for line in io.lines() do
	list = {next=list, value=line}
end
```

**一般化的列表初始化方式**

```lua
opnames = {["+"] = "add", ["-"] = "sub",["*"] = "mul", ["/"] = "div"}
-- 之前实用的方式都是对于这种一般化方式的简写
{x=0, y=0}   -- {["x"]=0, ["y"]=0}
{"red", "green", "blue"} --  {[1]="red", [2]="green", [3]="blue"}

-- 如何使得列表下标从0开始(不推荐，不能使用标准库函数)
days = {[0]="Sunday", "Monday", "Tuesday", "Wednesday","Thursday", "Friday", "Saturday"}
```

table中的逗号可以用分号替代

## 基本语法

#### 赋值语句

多赋值中注意

```lua
a,b,c=0  -- a=0,b=nil,c=nil
-- 应该用如下方式
a,b,c=0,0,0
```

#### 代码块

```lua
--显式指定一个代码块
do
    ..
end
```

#### 控制结构语句

##### repeat...until

```lua
-- 至少执行一次，当满足条件时循环结束
repeat
    ...
until condition
```

##### 数值for循环

```lua
for var=expr1,expr2,expr3 do
	...
end
```

1. 在循环开始之前三个表达式被计算一次
2. var是在循环内有效的，local
3. 循环过程中不要主动改变var的值。否则导致不可知状态

##### 泛型for循环

```lua
for i,v in ipairs(a) do 
    print(v) 
end
```

##### break和return

break 和 return 只能出现在 block 的结尾一句，基本不会有问题

为了调试，可以用`do return end`包装起来。

## 函数

函数调用，如果参数只有一个且参数类型为字符串或表构造时，可以省略括号。

```lua
print "hello world"
type {}
f {x=10,100}
```

#### table.unpack 函数

接收数组作为参数，返回数组的全部元素

```lua
-- 调用可变参数的可变函数
f=string.find
a={"xxz","x"}
f(unpack(a)) 

-- 手动实现unpack
function unpack(t, i)
    i = i or 1
    if t[i] then
        return t[i], unpack(t, i + 1)
    end
end
```

#### 可变参数

可变参数保存在`{...}`的表里面

```lua
g=function(...)
    arg={...}
    print(#arg)  --参数个数
    for v in ipairs(arg)  --遍历参数
    do
    	...
    end
end
```

select函数帮助处理可变参数

```lua
g=function(...) do -- 传入1，2，3，4，5
	selct(2,...)   -- 得到的仍是可变参数，但是从原参数的第2个开始。这里是2，3，4，5
	select(#,...)  -- 得到可变参数长度。这里是5
    x=selct(2,...) -- x=2.因为右侧参数个数更多，被丢弃
end
```

#### 命名参数

lua不支持命名参数，如果参数很多，用table传递参数是最好的办法

#### 闭包

内部函数实用外部的变量，既不是全局变量也不是局部变量，我们称作外部的局部变量（external local variable）或者 upvalue。  

闭包就是一个函数以及它的 upvalues  

```lua
function newCounter()
	local i = 0
	return function() -- anonymous function
		i = i + 1     --匿名函数中使用的i，就是upvalue
		return i
	end
end

function sortbygrade (names, grades)
    -- 这个例子里的匿名函数也是闭包，grades是upvalue
	table.sort(names, 
        function (n1, n2)
		return grades[n1] > grades[n2] 、
		end)
end
```

创建一个沙箱，来运行函数，例如

```lua
do
    local oldOpen = io.open
    io.open = function (filename, mode)
        if access_OK(filename, mode) then
            return oldOpen(filename, mode)
        else
            return nil, "access denied"
        end
    end
end
```

### 递归函数

```lua
local function ff(val)
    if val~=0 then
        return 101,ff(val-1)
    else
        return -1
    end
end
-- 这个例子失败，Lua编译时遇到ff(val-1)并不知道他是局部函数ff，
-- Lua会去查找是否有这样的全局函数ff。
local ff = function (val)
    if val~=0 then
        return 101,ff(val-1)
    else
        return -1
    end
end
-- 可以修改为这种方式
local ff
ff = function (val)
    if val~=0 then
        return 101,ff(val-1)
    else
        return -1
    end
end
```

#### lua可以正确处理尾递归调用

```lua
function foo (n)
	if n > 0 then 
        return foo(n - 1) 
    end
end
```

## 迭代器与泛型 for  

Lua 中常常使用函数来描述迭代器，每次调用该函数就返回集合的下一个元素 。

#### 泛性for

```lua
for <var-list> in <exp-list> do
	<body>
end
```

<var-list> 的第一个变量称为控制变量，其值为 nil 时循环结束。  

##### 执行过程

1. 初始化，计算 in 后面表达式的值，表达式应该返回范性 for 需要的三个值：迭代函数、状态常量、控制变量
2.  将状态常量和控制变量作为参数调用迭代函数（注意：对于 for 结构来说，状态常量没有用处，仅仅在初始化时获取他的值并传递给迭代函数）。  
3. 迭代函数返回的值赋给变量列表。
4. 如果返回的第一个值为 nil 循环结束，否则执行循环体。 

```lua
for var_1, ..., var_n in explist do block end
-- 等价于
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

#### 无状态的迭代器

迭代器本身取得下一个值，只需要状态常量和控制变量两个参数，其本身不保留状态。

```lua
function iter (a, i)  -- a状态常量  i控制变量
	i = i + 1
	local v = a[i]
	if v then
        return i, v
	end
end

function ipairs (a)
	return iter, a, 0
end
```

这个例子里面，每一次调用iter只需根据提供的a和i就能够计算出下一个值，因此无状态

#### 多状态的迭代器

迭代器本身是有状态的，需要额外的赋值变量来记录状态，一般有两种解决办法。

- 借助闭包来记录状态
- 将状态封装在table中，然后把table作为状态常量。相当于隐藏状态

通过一个例子，读取一个文件中的每一个单词的迭代器。

```lua
-- 通过闭包记录状态 
function allwords()
    local line = io.read()
    local pos = 1 
    return function()
        while line do -- repeat while there are lines
            local s, e = string.find(line, "%w+", pos)
            if s then -- found a word?
                pos = e + 1 -- next position is after this word
                return string.sub(line, s, e)-- return the word
            else
                line = io.read()-- word not found; try next line
                pos = 1 -- restart from first position
            end
        end
        return nil -- no more lines: end of traversal
    end
```

应该尽可能的写无状态的迭代器，因为这样循环的时候由 for 来保存状态，不需要创建对象花费的代价小；如果不能用无状态的迭代器实现，应尽可能使用闭包；尽可能不要使用 table 这种方式，因为创建闭包的代价要比创建 table 小，另外 Lua 处理闭包要比处理 table 速度快。 

## 编译运行

#### require函数

1. require 会搜索目录加载文件
2. require 会判断是否文件已经加载避免重复加载同一文件

#### C package

通过C语言为Lua写包。C语言需要先进行加载和连接，通过动态链接库机制实现

```lua
local path = "/usr/local/lua/lib/libluasocket.so"
 --指定路径和初始化函数，返回初始化函数作为 Lua 的一个函数
local f = loadlib(path, "luaopen_socket")  
f()  --真正打开苦，执行初始化函数
```

一般情况下我们期望二进制的发布库包含一个与前面代码段相似的 stub 文件，安装二进制库的时候可以随便放在某个目录，只需要修改 stub 文件对应二进制库的实际路径即可。将 stub 文件所在的目录加入到 LUA_PATH，这样设定后就可以使用 require 函数加载 C 库了。  

#### 错误

```lua
--  显式抛出错误
error("invalid input") 
-- 如果read失败，抛出错误。注意assert会先准备好两个参数，才判断。第二个参数一定会执行
n = assert(io.read("*number"), "invalid input")

-- error可以指定第二个参数，表示错误抛出的层级，1表示当前函数。2上一级函数。。。。。
error("invalid input",2)
```

对于程序逻辑上能够避免的异常，以抛出错误的方式处理之，否则返回错误代码。  

```lua
-- 打开文件的第二个参数就是错误信息。
file, msg = io.open(name, "r")
```

#### 异常和错误处理

##### pcall

pcall 在保护模式（protected mode）下执行函数内容，同时捕获所有的异常和错误。

pcall函数第一个返回值是函数的运行状态(true,false)，第二个返回值是pcall中函数的返回值。

状态为false时，第二个参数为错误信息(指error或者assert中指定的异常信息)。  

```lua
local _f = function()
    assert(nil, {error = 101})
end
local status, res = pcall(_f)
if status then
    print(res)
else
    print(res.error)
end
```

##### xcall

接收两个参数，目标函数和错误处理函数，pcall返回的时候，相关函数的堆栈信息已经销毁了，使用xcall解决

```lua
xpcall(foo(1),debug.traceback)
-- debug.debug 和 debug.traceback是两个用于输出调试信息的函数
```

## 协程

协程之间本质是单线程的，通过各个协程之间的协调分享CPU时间。例如，A调用了resume(B)之后，开始执行B，A就挂起了，直到B结束或者yield，才能轮到A。

```lua
co = coroutine.create(function)  -- 创建，返回类型为thread
coroutine.status(co)  -- 查看状态，有suspend、running、dead
coroutine.resume(co)  -- suspend转running
coroutine.yield()    --使得当前协程挂起    
    
-- 每一次resume，输出一个数字
local co = coroutine.create(function()
	 for var = 1, 10
     do
 		print(var)
     	coroutine.yield()
	end
end)
coroutine.resume(co)
coroutine.resume(co)
```

resume 运行在保护模式下，因此，如果协同程序内部存在错误， Lua 并不会抛出错误，而是将错误返回给 resume 函数。  即返回【false，msg】

#### resume和yield的参数传递

resume第一次调用的参数传递给目标函数，随后的参数都传到yield（前一次的导致协程阻塞的那个yield）作为yield的返回值

yield的参数都传给resume作为resume的返回值，如果目标函数执行完了，目标函数的返回值也会传递给resume作为resume的返回值

```lua
co = coroutine.create(function(x)
    print("x:", x)
    local y = coroutine.yield(555)
    print("y:", y)
    return 888
end)

print(coroutine.resume(co, 666))
print(coroutine.resume(co, 777))
--- 输出为
x:      666
true    555
y:      777
true    888
```

#### 生产者消费者问题

```lua
local producer = coroutine.create(function()
    local val = 0
    while true do
        print("producer:", val)
        coroutine.yield(val)    --生产一个val之后，挂起自身，将val传递出去
        val = val + 1
    end
end)
-- 消费者每次resume生产者获得一个新的值，生产者产生一个新值之后直接挂起
local function receiver()
    while true do
        local _, val = coroutine.resume(producer)  --第一个参数是bool
        print("reveiver:", val)
    end
end
receiver()--开始执行
```

#### 通过协同实现迭代器

迭代器函数中每次去resume一个生产者的协程，这样就能一个一个的获取元素了。

以获取一个list的全排列为例。

```lua
-- 提柜获取全排列，每找到一个则yield当前协程，传递出来
local function permgen(t, index)
    if index == #t then
        coroutine.yield(t)
    else
        for i = index, #t
        do
            t[i], t[index] = t[index], t[i]
            permgen(t, index + 1)
            t[i], t[index] = t[index], t[i]
        end
    end
end
-- 迭代器，每次resume协程，尝试获取一个值
local function perm(t)
    local co = coroutine.create(function()permgen(t, 1) end)
    return function()
        local _, v = coroutine.resume(co)
        return v
    end
end

for var in perm({1, 2, 3, 4})
do
    print(table.unpack(var))
end
```

##### coroutine.wrap(function)

创建协程并返回一个函数，执行这个函数就等价于resume协程。

wrap 中 resume的时候不会返回错误代码作为第一个返回结果，一旦有错误发生，将抛出错误 。

wrap缺少灵活性，没有办法知道 wrap所创建的协同的状态，也没有办法检查错误的发生。  

上述perm函数等价于

```lua
local function perm(t)
	return coroutine.wrap(function() permgen(t,1) end)
end
```



## Table

| 序号 | 方法 & 用途                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **table.concat (table [, sep [, start [, end]]]):** table.concat()函数列出参数中指定table的数组部分从start位置到end位置的所有元素, 元素间以指定的分隔符(sep)隔开。 |
| 2    | **table.insert (table, [pos,] value):**在table的数组部分指定位置(pos)插入值为value的一个元素. pos参数可选, 默认为数组部分末尾. |
| 3    | **table.remove (table [, pos])**返回table数组部分位于pos位置的元素. 其后的元素会被前移. pos参数可选, 默认为table长度, 即从最后一个元素删起。 |
| 4    | **table.sort (table [, comp])**                              |

#### 数组

table实现矩阵的时候，不需要考虑稀疏矩阵的问题，因为它存储的时候本身就是稀疏的

#### 队列

```lua
-- 为了不去污染全局名字空间，将全部操作封装在List里，实际上List就是一个类
List = {}
function List.new()
    return {first = 0, last = -1}
end
function List:push(value)
    self.last = self.last + 1
    self[self.last] = value
end

function List:pop()
    if self.first < self.last then
        error("empty")
    end
    local val = self[self.first]
    self[self.first] = nil  -- 务必记得删除元素
    self.first = self.first + 1
    return val
end
```

#### 持久化  

```lua
-- %q"选项，适用于字符串序列化，使用双引号表示字符串并且可以正确的处理包含引号和换行等特殊字符的字符串。
string.format("%q", o)
```

## metatable

算数运算包括`__add`、`__mul` 、`__sub(减)`、 `__div(除)`、**`__unm(负)`**、` __pow(幂)`、`__concat`、`__eq（等于）`，` __lt（小于）`，和`__le（小于等于）`  ,`__tostring  `

#### 保护你的metatable

```lua
-- 为matatable设置一个__metatable的域，即可保护这个metatable
s1 = setmetatable({},{__metatable = "not your business"})
print(getmetatable(s1)) --  not your business
setmetatable(s1, {})   --   stdin:1: cannot change protected metatable
```

#### 绕过元方法

```lua
-- 不受__index和__newindex的影响，直接操作原始表
rawget(t,k)  
rawset(t,k,v)
```

#### 有默认值的表

获取表中元素默认值是nil，也可以为其设置默认值

```lua
-- 将mt作为需要设置默认值的所有表的原表，在原表中调用字段`__`,然后每一个表分别指定自己的默认值
local mt = {__index = function(t)
    return t.__;
end}
function setDefault(t, default)
    t.__ = default
    return setmetatable(t, mt)
end
-- 使用
local t = setDefault({}, 120)
print(t.xyz)
```

#### 监控表

监控table中的访问和添加动作，为原始表提供一个空的代理表，这样每一个访问都会进入到监控

```lua
-- create private index
local index = {}
-- create metatable，建立监控
local mt = {
    __index = function(t, k)
        print("*access to element " .. tostring(k))
        return t[index][k]-- access the original table
    end,
    __newindex = function(t, k, v)
        print("*update of element " .. tostring(k) .. " to "
            .. tostring(v))
        t[index][k] = v -- update original table
    end
}
-- 对于表创建一层代理，将表放在proxy[index]中，index是一个私有空表，因此不会与正常的域产生冲突
-- 所有对于t的访问都会引导到mt中
function track(t)
    local proxy = {}
    proxy[index] = t
    setmetatable(proxy, mt)
    return proxy
end
```

#### 只读表

```lua
function ReadOnly(t)
    local proxy = {}
    local mt = {-- create metatable
        __index = t,
        __newindex = function(t, k, v)
            error("attempt to update a read-only table", 2)
        end
    }
    setmetatable(proxy, mt)
    return proxy
end
-- 使用
local t=ReadOnly({1,2,3})
t[4]=100
```

## 环境

#### 使用动态名字访问全局变量

例如一个变量的名字本身被保存在另一个变量中，运行时可以这样

```lua
val = _G[varname]
```

下列方法根据名字为嵌套作用域的变量赋值(获取值)

```lua
function getfield(f)
    local v = _G -- start with the table of globals
    for w in string.gfind(f, "[%w_]+") do
        v = v[w]
    end
    return v
end
function setfield(f, v)
    local t = _G -- start with the table of globals
    for w, d in string.gfind(f, "([%w_]+)(.?)") do
        if d == "." then -- not last field?
            t[w] = t[w] or {}-- create table if absent
            t = t[w]-- get the table
        else -- last field
            t[w] = v -- do the assignment
        end
    end
end
```

#### 非全局环境

`setfenv(1/2 , table)`函数来改变一个函数的环境。 setfenv 接受函数和新的环境作为参数。除了使用函数本身，还可以指定一个数字表示栈顶的活动函数。数字 1 代表当前函数，数字 2 代表调用当前函数的函数 。

一旦改变了环境，所有全局访问都使用新的表（而不是`_G`） 。

防止对全局环境污染的一个例子

```lua
--  这个例子里面，声明的所有全局变量都进入到newgt表中，不会污染_G
--  但是如果要使用库函数，仍然会从_G中获取
a = 1
local newgt = {} -- create new environment
setmetatable(newgt, {__index = _G})
setfenv(1, newgt) -- set it
print(a) --> 1
```

**上面都是lua5.1的做法**，从lua5.2开始，就不再有全局变量:

> 当你写a = 1的时候，其实被编译成 _ENV.a = 1

取而代之的是`_ENV` ，每一个chunk都有自己的`_ENV` ，在进入chunk的时候，lua将`_G`的值赋值给`_ENV` 

因此`_ENV` 就是标识了当前代码块的环境

## 面向对象



## String库

```lua
string.len(s)
string.rep(s,n)
string.lower(s)
string.upper(s)
string.sub(s,i,j)  -- i.j可以为负数，和py一样
string.char(97)  -- a
string.byte("abc",1)  -- 97  字符转数字
string.format(fmt,...)

--查找字符串，init表示起始查找位置，plain=true时关闭模式匹配，按普通字串处理pattern
string.find (s, pattern [, init [, plain]])

--替换字符串，n表示最多替换几次(否则全部换掉)
--放回值【替换后的字符串，替换了几次】
string.gsub(s,"pattern","replace",n) 
string.gfind
```

#### 模式匹配

lua不支持完整的正则匹配，自定义了一些模式匹配

```lua
. 任意字符
%a 字母
%c 控制字符
%d 数字
%l 小写字母
%p 标点字符
%s 空白符
%u 大写字母
%w 字母和数字
%x 十六进制数
%z 代表0的字符

其大写表示，是取非的意思。%A表示非字母
print(string.gsub("hello, up-down!", "%A", "."))
--> hello..up.down. 4

-- 以下为特殊字符
( ) . % + - * ? [ ^ $
'%'是转义字符（仅仅在模式匹配中有意义）
-- 统计元音字母的次数
_, nvow = string.gsub(text, "[AEIOUaeiou]", "")
-- 匹配（）之间的空白
%(%s*%)

- 和 * 都是匹配0次或多次，但是-最短匹配，*最长匹配
因为-是最短匹配，因此肯定不出现在模式串的最后(无意义)，可以出现在中间，通过后面的模式限定
-- 例如，用于匹配C语言的注释
"/%*_-%*/"

-- 构造一个包含至少六个(字母数字标点空格)的串，且以0值结尾
-- 0值一般是在二进制文件中才可能出现，就是字节意义上的0
local validchars = "[%w%p%s]"
local pattern = string.rep(validchars, 6) .. "+%z"
```

捕获(其实就是分组)

```lua
pair = "name = Anna"
-- 返回起止位置，以及两个捕获的值(这里只关心捕获)
_, _, key, value = string.find(pair, "(%a+)%s*=%s*(%a+)")
print(key, value) --> name Anna
```

## IO库

#### 简单模式（simple model）

拥有一个当前输入文件和一个当前输出文件，并且提供针对这些文件相关的操作。

```  lua
io.input()   -- 指定输入，或返回输入(不需要参数)
io.output()

io.read()
io.write()
--io.write() 和 print()
write 不附加任何额外的字符到输出中去，例如制表符，换行符等等。还有 write 函数是使用当前输出文件，而 print 始终使用标准输出。另外print函数会自动调用参数的tostring方法，所以可以显示出表（tables）函数（functions）和 nil

-- io.read()的参数
"*all" 读取整个文件
"*line" 读取下一行
"*number" 从串中转换出一个数值
num 读取 num 个字符到串

```

#### 完全模式（complete model）

使用外部的文件句柄来实现。它以一种面对对象的形式，将所有的文件操作定义为文件句柄的方法。  

```lua
-- 返回句柄，或者[nil,错误信息]
f=io.open(filename,"r")  -- r w a/rb wb ab
f:read("*all")
f:write(XXXX)
f:close()

io.stderr:write(msg)
```

#### 读取一定字节的数据

```lua
lines, rest = f:read(9, "*line")
-- 读取9字节的数据，放入lines
-- 若lines没有到行结尾，则将读到的最后一行的剩余部分写到rest中
```



