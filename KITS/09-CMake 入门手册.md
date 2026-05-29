# 概述

CMake 是一个开源、跨平台的自动化构建工具，通过 CMake 可以轻松管理项目   
- 操作透明细腻
- 支持 C++ 现代化编译器和工具链
- 支持跨平台
- 支持主流 IDE

## 流程

1. 预处理（-E）：宏替代
2. 编译（-S）：高级语言->汇编语言
3. 汇编（-C）：汇编语言->机器码
4. 链接：多个二进制文件链接生成可执行文件

## CMake 执行流程

1. 编写 CMakeLists.txt 文件
   -  cmake_minimum_required(version 3.20) # 最小版本
   -  project(Hello) # 项目名
   -  add_executable(Hello hello.cpp) # 源文件
2. cmake -B build
   - 创建 build 目录并生成文件
3. cmake --bulid build
   - 生成项目

# 命令

```
cmake_minimum_required(VERSION 3.20) # 指定版本
message("str")  # 输出信息
message(${CMAKE_VERSION}) # 获取 CMake 信息

set(varname <value..>)  # 设置变量
${varname}              # 获取变量值
unset(varname)          # 取消设置

include(file|module)    # 引入头文件

list(LENGTH <list> <out-var>) # 获取长度
list(GET <list> <element index> [<index> ...] <out-var>)
list(JOIN <list> <glue> <out-var>)
list(SUBLIST <list> <begin> <length> <out-var>)

list(FIND <list> <value> <out-var>) # 查找元素

Modification
  list(APPEND <list> [<element>...])
  list(FILTER <list> {INCLUDE | EXCLUDE} REGEX <regex>)
  list(INSERT <list> <index> [<element>...])
  list(POP_BACK <list> [<out-var>...])
  list(POP_FRONT <list> [<out-var>...])
  list(PREPEND <list> [<element>...])
  list(REMOVE_ITEM <list> <value>...)
  list(REMOVE_AT <list> <index>...)
  list(REMOVE_DUPLICATES <list>)
  list(TRANSFORM <list> <ACTION> [...])

Ordering
  list(REVERSE <list>)
  list(SORT <list> [...])
```

## 控制结构

```
if()...elseif()...else()...endif()
foreach()...endforeach
while()...endwhile()
```