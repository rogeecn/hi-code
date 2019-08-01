---
title: Lua学习笔记（一）
date: 2018-03-19 00:00:00
tags: ["LUA"]
abbrlink: lua-lear-note-section-1
img: ""
comments: false
---

## Lua 基础数据类型
函数  `type` 能够返回一个值或一个变量所属的类型。
```
print(type("hello world")) -->output:string
print(type(print))         -->output:function
print(type(true))          -->output:boolean
print(type(360.0))         -->output:number
print(type(nil))           -->output:nil
```

| 类型  | 说明  | 示例
| :------------ | :------------ | :------------ |
| nil  | 空  | |
| number |  数字 | |
| string | 字符串 | 'hello', "hello"， [["hello",'world']]，[=[string [[]].]=]  |
| table | 表 ||
| function | 函数 ||


### table 
Table 类型实现了一种抽象的“关联数组”。“关联数组”是一种具有特殊索引方式的数组，索引通常是字符串（string）或者 number 类型，但也可以是除 nil 以外的任意类型的值。
```
local corp = {
    web = "www.google.com",   --索引为字符串，key = "web",
                              --            value = "www.google.com"
    telephone = "12345678",   --索引为字符串
    staff = {"Jack", "Scott", "Gary"}, --索引为字符串，值也是一个表
    100876,              --相当于 [1] = 100876，此时索引为数字
                         --      key = 1, value = 100876
    100191,              --相当于 [2] = 100191，此时索引为数字
    [10] = 360,          --直接把数字索引给出
    ["city"] = "Beijing" --索引为字符串
}

print(corp.web)               -->output:www.google.com
print(corp["telephone"])      -->output:12345678
print(corp[2])                -->output:100191
print(corp["city"])           -->output:"Beijing"
print(corp.staff[1])          -->output:Jack
print(corp[10])               -->output:360
```
###  function
```
local function foo()
    print("in the function")
    --dosomething()
    local x = 10
    local y = 20
    return x + y
end

local a = foo    --把函数赋给变量

print(a())

--output:
in the function
30
```

## 表达式
### 算术运算符
| 算术运算符  | 说明  |
| :------------ | :------------ |
|  + | 加法  |
|  - | 减法  |
|  * | 乘法  |
|  / | 除法  |
|  ^ | 指数  |
|  % | 取模  |

### 关系运算符
| 关系运算符  | 说明  |
| :------------ | :------------ |
|  < | 小于  |
|  > | 大于  |
|  <= | 小于等于  |
|  >= | 大于等于  |
|  == | 等于  |
|  ~= | 不等于  |

### 逻辑运算符
| 关系运算符  | 说明  |
| :------------ | :------------ |
|  and | 与 |
| or | 或 |
| not | 非 |

### 字符串连接
在 Lua 中连接两个字符串，可以使用操作符“..”（两个点）。如果其任意一个操作数是数字的话，Lua 会将这个数字转换成字符串.
```
"Hello " .. "World"
```

### 运算优先级
若不确定某些操作符的优先级，就应显示地用括号来指定运算顺序。这样做还可以提高代码的可读性。
| 优先级  |
| :------------ |
| ^  |
| not   # - |
| *   /   % |
| +   - |
| .. |
| < > <=  >=  ==  ~= |
| and |
| or |

## 控制结构

### if/else
```lua
score = 90
if score == 100 then
    print("Very good!Your score is 100")
elseif score >= 60 then
    print("Congratulations, you have passed it,your score greater or equal to 60")
-- 此处可以添加多个elseif
else
    print("Sorry, you do not pass the exam! ")
end
```
### while
Lua 并没有像许多其他语言那样提供类似 **continue** 这样的控制语句用来立即进入下一个循环迭代
```
local t = {1, 3, 5, 8, 11, 18, 21}

local i
for i, v in ipairs(t) do
    if 11 == v then
        print("index[" .. i .. "] have right value[11]")
        break
    end
end
```
### repeat
```
x = 10
repeat
    print(x)
until false
```

### for
```
for var = begin, finish, step do
    --body
end
```
### for 泛型
```
-- 打印数组a的所有值
local a = {"a", "b", "c", "d"}
for i, v in ipairs(a) do
  print("index:", i, " value:", v)
end

-- output:
index:  1  value: a
index:  2  value: b
index:  3  value: c
index:  4  value: d
```
### break
### return

## 函数
```
function function_name (arc)  -- arc 表示参数列表，函数的参数列表可以为空
   -- body
end


function_name = function (arc)
  -- body
end

function foo.bar(a, b, c)
    -- body ...
end

-- 由于全局变量一般会污染全局名字空间，同时也有性能损耗（即查询全局环境表的开销），因此我们应当尽量使用“局部函数”，其记法是类似的，只是开头加上 local 修饰符：
local function function_name (arc)
  -- body
end

--- 变长参数
local function func( ... )                -- 形参为 ... ,表示函数采用变长参数
   local temp = {...}                     -- 访问的时候也要使用 ...
   local ans = table.concat(temp, " ")    -- 使用 table.concat 库函数对数
                                          -- 组内容使用 " " 拼接成字符串。
   print(ans)
end

```
Lua 函数的参数大部分是按值传递的,如果形参为TABLE，那么会按**引用传递**。
### 函数返回值
```
local s, e = string.find("hello world", "llo")
```

### 全动态函数调用
调用回调函数，并把一个数组参数作为回调函数的参数。
```
local args = {...} or {}
method_name(unpack(args, 1, table.maxn(args)))
```
#### 使用场景
如果你的实参 `table` 中确定没有 `nil` 空洞，则可以简化为
```
method_name(unpack(args))
```
- 你要调用的函数参数是未知的；
- 函数的实际参数的类型和数目也都是未知的。
伪代码
```
add_task(end_time, callback, params)

if os.time() >= endTime then
    callback(unpack(params, 1, table.maxn(params)))
end
```
值得一提的是，unpack 内建函数还不能为 LuaJIT 所 JIT 编译，因此这种用法总是会被解释执行。对性能敏感的代码路径应避免这种用法。
#### 小试牛刀
```
local function run(x, y)
    print('run', x, y)
end

local function attack(targetId)
    print('targetId', targetId)
end

local function do_action(method, ...)
    local args = {...} or {}
    method(unpack(args, 1, table.maxn(args)))
end

do_action(run, 1, 2)         -- output: run 1 2
do_action(attack, 1111)      -- output: targetId    1111
```

### 参考
[OpenResty最佳实践](https://moonbingbing.gitbooks.io "OpenResty最佳实践")
