---
title: Lua学习笔记（二）
date: 2018-03-20 00:00:00
tags: ["LUA"]
abbrlink: lua-lear-note-section-2
img: ""
comments: false
---

## 模块

模块文件`my.lua`
```
local foo={}

local function getname()
    return "Lucy"
end

function foo.greeting()
    print("hello " .. getname())
end

return foo
```
调用
```
local fp = require("my")
fp.greeting()
```



## 元表
在 Lua 5.1 语言中，元表 (metatable) 的表现行为类似于 C++ 语言中的操作符重载，例如我们可以重载 "__add" 元方法 (metamethod)，来计算两个 Lua 数组的并集；或者重载 "__index" 方法，来定义我们自己的 Hash 函数。Lua 提供了两个十分重要的用来处理元表的方法，如下：
- setmetatable(table, metatable)：此方法用于为一个表设置元表。
- getmetatable(table)：此方法用于获取表的元表对象。
```
local mytable = {}
local mymetatable = {}
setmetatable(mytable, mymetatable)
```
### 元表操作符重载
通过重载 "__add" 元方法来计算集合的并集实例：
```
local set1 = {10, 20, 30}   -- 集合
local set2 = {20, 40, 50}   -- 集合

-- 将用于重载__add的函数，注意第一个参数是self
local union = function (self, another)
    local set = {}
    local result = {}

    -- 利用数组来确保集合的互异性
    for i, j in pairs(self) do set[j] = true end
    for i, j in pairs(another) do set[j] = true end

    -- 加入结果集合
    for i, j in pairs(set) do table.insert(result, i) end
    return result
end
setmetatable(set1, {__add = union}) -- 重载 set1 表的 __add 元方法

local set3 = set1 + set2
for _, j in pairs(set3) do
    io.write(j.." ")               -->output：30 50 20 40 10
end
```

### 其它操作符
| 元方法 | 含义 |
| ------------ | ------------ |
| __add | + |
| __sub | - |
| __mul | * |
| __div | / |
| __mod | % |
| __pow | ^ |
| __unm | 一元 - 操作 |
| __concat | .. |
| __len | # |
| __eq| == |
| __lt | < |
| __le | <= |
除了操作符之外，如下元方法也可以被重载
|元方法 | 含义 |
| ------------ | ------------ |
| __index | 取下标操作用于访问 table[key] |
| __newindex | 赋值给指定下标 table[key] = value |
| __tostring | 转换成字符串 |
| __call | 当 Lua 调用一个值时调用 |
| __mode | 用于弱表(week table) |
| __metatable | 用于保护metatable不被访问 |
#### __index 元方法
```
mytable = setmetatable({key1 = "value1"},   --原始表
  {__index = function(self, key)            --重载函数
    if key == "key2" then
      return "metatablevalue"
    end
  end
})

print(mytable.key1,mytable.key2)  --> output：value1 metatablevalue
```
#### __tostring 元方法
```
arr = {1, 2, 3, 4}
arr = setmetatable(arr, {__tostring = function (self)
    local result = '{'
    local sep = ''
    for _, i in pairs(self) do
        result = result ..sep .. i
        sep = ', '
    end
    result = result .. '}'
    return result
end})
print(arr)  --> {1, 2, 3, 4}
```
#### __call 元方法
```
functor = {}
function func1(self, arg)
  print ("called from", arg)
end

setmetatable(functor, {__call = func1})

functor("functor")  --> called from functor
print(functor)      --> output：0x00076fc8 （后面这串数字可能不一样）
```
#### __metatable 元方法
```
Object = setmetatable({}, {__metatable = "You cannot access here"})

print(getmetatable(Object)) --> You cannot access here
setmetatable(Object, {})    --> 引发编译器报错
```

## Lua 面向对象编程
### 类
类文件`account.lua`
```
local _M = {}

local mt = { __index = _M }

function _M.deposit (self, v)
    self.balance = self.balance + v
end

function _M.withdraw (self, v)
    if self.balance > v then
        self.balance = self.balance - v
    else
        error("insufficient funds")
    end
end

function _M.new (self, balance)
    balance = balance or 0
    return setmetatable({balance = balance}, mt)
end

return _M
```
调用方法
```
local account = require("account")

local a = account:new()
a:deposit(100)

local b = account:new()
b:deposit(50)

print(a.balance)  --> output: 100
print(b.balance)  --> output: 50
```
### 继承
```
--------- s_base.lua
local _M = {}

local mt = { __index = _M }

function _M.upper (s)
    return string.upper(s)
end

return _M

---------- s_more.lua
local s_base = require("s_base")

local _M = {}
_M = setmetatable(_M, { __index = s_base })


function _M.lower (s)
    return string.lower(s)
end

return _M

---------- test.lua
local s_more = require("s_more")

print(s_more.upper("Hello"))   -- output: HELLO
print(s_more.lower("Hello"))   -- output: hello
```

### 成员私有性
```
function newAccount (initialBalance)
    local self = {balance = initialBalance}
    local withdraw = function (v)
        self.balance = self.balance - v
    end
    local deposit = function (v)
        self.balance = self.balance + v
    end
    local getBalance = function () return self.balance end
    return {
        withdraw = withdraw,
        deposit = deposit,
        getBalance = getBalance
    }
end

a = newAccount(100)
a.deposit(100)
print(a.getBalance()) --> 200
print(a.balance)      --> nil
```
## 判断数组大小
以第一个 nil 元素来判断数组结束,不要在 Lua 的 table 中使用 nil 值，如果一个元素要删除，直接 remove，不要用 nil 去代替。
```
local tblTest1 = { 1, a = 2, 3 }
print("Test1 " .. #(tblTest1))
```

## 非空判断
table 不参直接 `=nil`来判定
```
function isTableEmpty(t)
    return t == nil or next(t) == nil
end
```
