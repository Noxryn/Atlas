# 变量

Python 中"变量名贴在值上"而不是"值存储在变量中"
- 变量使用前必须赋值
- 变量名可以为字母、数字、下划线，但不能以数字开头
- 字母区分大小写
  
原始字符串：`r"str text"`

# 数据类型

整型 `int`  
浮点数 `float`  
字符串 `str`
获得数据类型函数 `type()` or `isinstance(变量,类型)`  

数字可以用下划线分隔 `1_000 = 1000`

## 列表

数组

语法：`arr = [val1,val2,val3,val4]`  
- 变量值类型可以混合

```
append(val) # 添加元素
extend([val]) # 以列表扩展元素
insert(addr,val) # 插入元素

remove(val) # 删除指定元素
del arr[index] # 删除语句
pop(<index>) # 弹出元素

arr[start:stop] # 列表分片

dir(arr) # 操作查看

sort() # 排序 <reverse> - 排序顺序
sorted() # 临时排序

reverse() # 倒序

len() # 获取长度

```

## 元组

语法：`tuplel = (val,val)`

## 字符串

```
titile() # 首字符大写
upper() # 字符大写
lower() # 字符小写

"{0} {1}".format(val1,val2) # 格式化
f"{val1}{val2}"

restrip() # 删除空白
```
## 字典
dict = {key:val}
```
get(key, <"return text">)
```

# 操作符
| 功能               | 操作符            |
| ------------------ | ----------------- |
| 幂运算             | `**`              |
| 正负号             | `+x -x`           |
| 加减乘除           | `+ - * /`         |
| 进位除法，舍弃精度 | `//`              |
| 比较操作符         | `< <= > >= == !=` |
| 逻辑操作符         | `not or and`      |

# 条件分支

## if 

```
if 条件:

elif 条件:

else:

```
## 三元运算符

```
真 if 条件 ? 假
```

## 断言

条件为假时，程序自动奔溃并抛出 `AssertionError` 异常    
语法： `assert 条件`


# 循环分支

## while 

```
while 条件:
    循环体
```

## for

```
for 目标 in 表达式
    循环体
```

## range()

语法：`range ([strat,] stop [, step=1])`
- 生成从 start 开始到 stop 参数值结束的数字序列  
  
  max min sum