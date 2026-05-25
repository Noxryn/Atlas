# 什么是 makefile?

现代 IDE 中一般集成了 CMake 等编译工具，使程序员无需关心源码的链接问题。但像在 Linux 环境下进行开发，不会写 makefile，就不具备完成大型工程的能力  

相比 makefile，更熟悉的可能是 make。make 是一个用于解释 makefile 指令的命令工具。 使用 make 可以自动编译工程项目，提高软件开发效率。make 命令执行时，需要一个 Makefile 文件，以告诉 make 命令需要怎么样的去编译和链接程序

makefile 文件中定义了一系列的规则来指定程序项目中源文件的编译顺序或进行更复杂的功能操作

# Makefile 使用简介

## makefile 编译链接规则

- 如果这个工程没有编译过，那么我们的所有文件都要编译并被链接。
- 如果这个工程的某几个文件被修改，那么我们只编译被修改的文件，并链接目标程序。
- 如果这个工程的头文件被改变了，那么我们需要编译所有引用了这几个头文件的文件，并链接目标程序。

## Makefile 组成

Makefile 由显示规则、隐晦规则、变量定义、文件指示和注释五部分组成。  
1. 显示规则：说明生成一个或多个目标文件(目标文件、依赖文件、执行命令)。  
2. 隐晦规则：利用自动推导功能简化书写。  
3. 变量定义：宏 , 变量名 obj，使用方法 `${obj}`
4. 文件指示：Makefile 文件中引用另一 Makefile; 指定执行 Makefile 执行内容; 定义多行命令。  
5. 注释。Makefiel 中只有行注释，以`#`注释。  

## 文件名

make 命令会在当前目录下寻找文件名为  `GNUmakefile` 、`makefile` 和 `Makefile` 的文件 我们也可以使用选项 
`make -f %{filename} / make --file ${filename}` 指定 Makefile 文件  

## 引用其它 Makefile

`[-] include ${Makefile filename}` 命令将其它 Makefile 包含到文件当前位置   
- 首先在当前目录下寻找
- 寻找 `-I` 或 `--include-dir` 参数指定目录
- 寻找目录 `<prefix>/include` (/us/local/bin 或 /usr/include)  

注：命令前使用减号表示无论出现任何错误都继续执行  

## 工作原理

1. 执行 make 命令
2. make 在当前目录下查找文件名为 "Makefile" 或 "makefile" 的文件
3. 读入被 `include` 的其它 Makefile
4. 初始化变量
5. 推导隐晦规则，并分析所以规则
6. 为所有目标文件创建依赖关系链
7. 根据依赖关系确定目标生成  
8. 执行文件中的命令。存在目标文件则跳过，不存在则生成，存在但依赖文件更新则重新生成。  
9. make 一层一层地查找文件间的依赖关系，直至生成目标文件。   

# 书写规则

- 依赖关系
- 目标生成方法

## 示例

```
foo.o : foo.c defs.h # 依赖关系
    cc -c -g foo.c   # 生成方法
```

## 语法规则  

```
target:prerequisites
    command
```

- target 即目标文件，支持中间目标文件、执行文件和标签。  
- prerequisites 即生成 target 所需要的文件或目标。  
- command 即 make 需要执行的命令。
    
target 目标文件依赖于 prerequisites 中的文件，其生成规则定义在 command 中。即 prerequisites 中任一文件比 target 文件新则执行 command 命令。

注：makefile 中以 `<Tab>` 为缩进，`#` 为注释。
>可以在 makefile 文件的开始定义 `.RECIPEPREFIX = >` ，那么后面所有的命令前缀将使用 `>`， 而不再是 `Tab` ，直到 `.RECIPEPREFIX` 被赋给其他值

## 通配符

| 通配符 | 作用           |
| ------ | --------------|
| `~`    | 当前 HOME 目录 |
| `*`    | 任意字符匹配   |
| `？`   | 单个字符匹配   |

## 文件搜寻

在一些大的工程中，有大量的源文件，分类存放在不同的目录中。当 make 需要去找寻文件的依赖关系时，需要把一个路径告诉 make，让make 在自动去找  

1. VPATH  变量
   
指定目录中寻找文件 `VPATH = ${dirname1} : ${dirname2}`     

2. vpath 关键字

   1. 为符合模式 `<pattern>` 的文件指定搜索目录 
        `vpath <pattern> <directories>` 
   2. 清除符合模式 `<pattern>` 的文件搜索目录
         `vpath <pattern>`
   3. 清除所有设置的文件目录`vpath`

- ***vpath 关键字中 `<pattern>` 使用 `%` 匹配字符***
- 连续使用 vpath 语句指定不同搜索策略，相同 `<pattern>` 则顺序执行搜索

示例：
```
vpath %.c ../src  # vpath <匹配文件> <搜索目录>
```

## 伪目标

示例：
```
.PHONY:clean # 防止伪目标和变量重名，要运行 {clean} 必须 make {clean}
clean:
    -rm ${obj} target
```
伪目标置于首位作为默认目标，伪目标的特性是总是被执行的，基于此特性可以执行全部生成文件  
```
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
    cc -o prog1 prog1.o utils.o
prog2 : prog2.o
    cc -o prog2 prog2.o
prog3 : prog3.o sort.o utils.o
    cc -o prog3 prog3.o sort.o utils.o
```
伪目标作为依赖，实现不同功能
```
.PHONY: cleanall cleanobj cleandiff
cleanall : cleanobj cleandiff
    rm program
cleanobj :
    rm *.o
cleandiff :
    rm *.dif
```
## 静态模式

多目标：Makefile 的规则中的目标可以不止一个

```
bigoutput littleoutput : text.g
    generate text.g -$(subst output,,$@) > $@
```
上述规则等价于：
```
bigoutput : text.g
    generate text.g -big > bigoutput
littleoutput : text.g
    generate text.g -little > littleoutput
```

静态模式可以更加容易地定义多目标的规则
```
<targets ...>: <target-pattern>: <prereq-patterns ...>
    <commands>
```
- targets 定义了一系列的目标文件，可以有通配符。是目标的一个集合。
- target-parrtern 是指明了targets 的模式，也就是的目标集模式。
- prereq-parrterns 是目标的依赖模式，它对target-parrtern 形成的模式再进行一次依赖目标的定义

例:
```
objects = foo.o bar.o
all: $(objects)
$(objects): %.o: %.c
    $(CC) -c $(CFLAGS) $< -o $@
```
等价于
```
foo.o : foo.c
    $(CC) -c $(CFLAGS) foo.c -o foo.o
bar.o : bar.c
    $(CC) -c $(CFLAGS) bar.c -o bar.o
```

## 自动依赖关系

C/C++ 编译器都支持一个 `-M` 的选项，即自动找寻源文件中包含的头文件，并生成一个依赖关系。如果使用 GNU 的 C/C++ 编译器用 `-MM` 参数  

GNU 组织建议把编译器为每一个源文件的自动生成的依赖关系放到一个
文件中，为每一个 name.c 的文件都生成一个 name.d 的 Makefile 文件，.d 文件中就存放对应 .c 文件的依赖关系  

规则：
```
.RECIPEPREFIX = >
%.d: %.c
>  @set -e; rm -f $@; \
    $(CC) -M $(CPPFLAGS) $< > $@.$$$$; \
    sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
    rm -f $@.$$$$
```

```
include $(sources:.c=.d) #.c 替换成 .d 
```

# 书写命令

## 显示命令

通常，make 会把其要执行的命令行在命令执行前输出到屏幕上。当用 @ 字符在命令行前，那么，这个命令将不被 make 显示出来  

如果 make 执行时，带入 make 参数 -n 或 --just-print ，那么其只是显示命令，但不会执行命令

make 参数 -s 或 --silent 或 --quiet 则是全面禁止命令的显示  

注：要让上一条命令的结果应用在下一条命令时，应该使用分号分隔这两条命令
```
cd /home/hchen; pwd
```
## 命令出错

make 加上 -i 或是 --ignore-errors 参数，所有命令都会忽略错误 

make 的参数的是 -k 或是 --keep-going ，这个参数的意思是，如果某规则中的命令出错了，那么就终止该规则的执行，但继续执行其它规则  

## 嵌套执行 make

在一些大的工程中，会把不同模块或是不同功能的源文件放在不同的目录中，可以在每个目录中都书写一个该目录的 Makefile ，这有利于让 Makefile 变得更加地简洁，但很难维护，这个技术对于模块编译和分段编译有着非常大的好处。  

```
.RECIPEPREFIX = >
subsystem:
>  cd subdir && $(MAKE)
```
其等价于：
```
.RECIPEPREFIX = >
subsystem:
>  $(MAKE) -C subdir
```
定义 $(MAKE) 宏变量的意思是，make 可能需要一些参数，所以定义成一个变量比较利于维护。这两个例子的都是先进入"subdir" 目录，然后执行make命令。 

总控 Makefile 的变量可以传递到下级的 Makefile 中，但是不会覆盖下层的 Makefile 中所定义的变量，除非指定了 -e 参数。

要传递变量到下级Makefile中，可以使用:   
`export <variable ...>;`

不想让某些变量传递到下级 Makefile，可以声明:  
`unexport <variable ...>;`

如：
```
export variable = value
```
其等价于：
```
variable = value
export variable
```
其等价于：
```
export variable := value
```
其等价于：
```
variable := value
export variable
```
export 后面什么也不用跟，表示传递所有的变量  

注：有两个变量，一个是 SHELL ，一个是 MAKEFLAGS ，这两个变量总是要传递到下层 Makefile中，特别是 MAKEFLAGS 变量  

如果你不想往下层传递参数可以：
```
.RECIPEPREFIX = >
subsystem:
>  cd subdir && $(MAKE) MAKEFLAGS=
``` 
 -w 或是 --print-directory 会在make的过程中输出一些信息  

当使用 -C 参数来指定 make 下层 Makefile 时， -w 会被自动打开的。  

如果参数中有 -s （ --slient ）或是 --no-print-directory ，那么， -w 总是失效的

## 定义命令包

如果 Makefile 中出现一些相同命令序列，可以为这些相同的命令序列定义一个变量。   
定义这种命令序列的语法以 define 开始，以 endef 结束，如:
```
define run-yacc  # 定义 命令包 run-yacc
yacc $(firstword $^)
mv y.tab.c $@
endef

$(run-yacc) # 执行

```
make在执行命令包时，命令包中的每个命令会被依次独立执行  

# 变量

- 变量的命名字可以包含字符、数字，下划线（可以是数字开头）
- 不应该含有 : 、 # 、 = 或是空字符（空格、回车等）  
- 变量是大小写敏感的
- 有一些变量是很奇怪字串，如 $< 、 $@ 等，是自动化变量
- 变量在声明时需要给予初值
- 在使用时，需要给在变量名前加上 $ 符号，用{} 把变量给包括起来
- 使用真实的 $ 字符，需要用 $$ 来表示
- 变量是可以使用后面的变量来定义

```
foo = $(bar) # ${foo} 为 huh
bar = $(ugh)
ugh = Huh

all:
>  echo $(foo)
```

## Makefile 赋值

| 赋值 | 说明                               |
| ---- | ---------------------------------- |
| =    | 最后赋值                           |
| :=   | 覆盖之前的值, 不能使用后面的变量值 |
| ?=   | 未赋值则赋值                       |
| +=   | 添加后面值                         |

## 变量替换

1. 变量值  
替换变量中的共有的部分，其格式是 `$(var:a=b)`
2. 静态技术  
   依赖于被替换字串中的有相同的模式，其格式是 `$(var:%a=%b)`

注：变量如果是通过 make 的命令行参数设置的，那么 Makefile 中对这个变量的赋值会被忽略。想在 Makefile 中设置这类参数的值，可以使用"override"指示符。其语法是:
```
override <variable>; = <value>;
override <variable>; := <value>;
override <variable>; += <more text>; # 追加
```
## 目标变量

作用范围只在这条规则以及连带规则中，所以其值也只在作用范围内有效。而不会影响规则链以外的全局变量的值  
```
<target ...> : <variable-assignment>;
<target ...> : overide <variable-assignment>
```
`<variable-assignment>;`可以是前面讲过的各种赋值表达式，如 = 、 := 、 += 或是 ?= 

第二个语法是针对于 make 命令行带入的变量，或是系统环境变量

示例：
```
prog : CFLAGS =-g
prog : prog.o foo.o
    $(CC) $(CFLAGS) prog.o foo.o bar.o
prog.o : prog.c
    $(CC) $(CFLAGS) prog.c
foo.o : foo.c
    $(CC) $(CFLAGS) foo.c

# $(CFLAGS) 的值都是-g
```

# 条件判断

语法：
```
ifeq(val , check) 
    # ...
else
    # ...
endif
```
## 条件关键字

- ifeq   arg1 和 arg2 相同
- ifneq  arg1 和 arg2 不同
- ifdef  值非空
- ifndef 值为空

# 函数

## 调用语法

```
$(<function> <arguments>)
```
- <function> 就是函数名
- <arguments> 为函数的参数， 参数间以逗号 `,` 分隔
- 函数名和参数之间以"空格"分隔
- 函数调用以 `$` 开头，以圆括号或花括号把函数名和参数括起

## 字符串处理函数

### subst

`$(subst <from>,<to>,<text>)`
- 名称：字符串替换函数
- 功能：把字串 <text> 中的 <from> 字符串替换成 <to> 。
- 返回：函数返回被替换过后的字符串。
  
### patsubst

`$(patsubst <pattern>,<replacement>,<text>)`
- 名称：模式字符串替换函数。
- 功能：查找 <text> 中的单词（单词以"空格"、"Tab"或"回车""换行"分隔）是否符合模式 <pattern> ，如果匹配的话，则以 <replacement> 替换。这里，<pattern> 可以 包括通配符 % ，表示任意长度的字串。如果 <replacement> 中也包含 % ，那么， <replacement> 中的这个 % 将是 <pattern> 中的那个 % 所代表的字串。 （可以用 \ 来转义，以 \% 来表示真实含义的 % 字符）
- 返回：函数返回被替换过后的字符串。

### strip

`$(strip <string>)`
- 名称：去空格函数。
- 功能：去掉 <string> 字串中**开头**和**结尾**的空字符。
- 返回：返回被去掉空格的字符串值。

### findstring

`$(findstring <find>,<in>)`
- 名称：查找字符串函数
- 功能：在字串 <in> 中查找 <find> 字串。
- 返回：如果找到，那么返回 <find> ，否则返回空字符串。
  
### filter

`$(filter <pattern...>,<text>)`
- 名称：过滤函数
- 功能：以 <pattern> 模式过滤 <text> 字符串中的单词，保留符合模式 <pattern> 的单词。可以有多个模式。
- 返回：返回符合模式 <pattern> 的字串。

### filter-out

`$(filter-out <pattern...>,<text>)`
- 名称：反过滤函数
- 功能：以 <pattern> 模式过滤 <text> 字符串中的单词，去除符合模式 <pattern> 的单词。可以有多个模式。
= 返回：返回不符合模式 <pattern> 的字串。

### sort

`$(sort <list>)`
- 名称：排序函数
- 功能：给字符串 <list> 中的单词排序（升序）。
- 返回：返回排序后的字符串。

### word

`$(word <n>,<text>)`
- 名称：取单词函数
- 功能：取字符串 <text> 中第 <n> 个单词。（从一开始）
- 返回：返回字符串 <text> 中第 <n> 个单词。如果 <n> 比 <text> 中的 单词数要大，那么返回空字符串。

### wordlist

`$(wordlist <ss>,<e>,<text>)`
- 名称：取单词串函数
- 功能：从字符串 <text> 中取从 <ss> 开始到 <e> 的单词串。 <ss> 和 <e> 是一个数字。
- 返回：返回字符串 <text> 中从 <ss> 到 <e> 的单词字串。如果 <ss> 比 <text> 中的单词数要大，那么返回空字符串。如果 <e> 大于 <text> 的单词数， 那么返回从 <ss> 开始，到 <text> 结束的单词串。

### words

`$(words <text>)`
- 名称：单词个数统计函数
- 功能：统计 <text> 中字符串中的单词个数。
- 返回：返回 <text> 中的单词数。

注：取 <text> 中最后的一个单词，可以： $(word $(words <text>),<text>) 

### firstword

`$(firstword <text>)`
- 名称：首单词函数
- 功能：取字符串 <text> 中的第一个单词。
- 返回：返回字符串 <text> 的第一个单词。

备注：这个函数可以用 word 函数来实现： $(word 1,<text>) 

## 文件名操作函数

### wildcard

`$(wildcard <pattern>)`
- 名称：通配符函数
- 功能：从当前目录下获得所有符合模式的文件
- 匹配的源文件

### dir

`$(dir <names...>)`
- 名称：取目录函数
- 功能：从文件名序列 <names> 中取出目录部分。目录部分是指最后一个反斜杠（ / ）之前 的部分。如果没有反斜杠，那么返回 ./ 
- 返回：返回文件名序列 <names> 的目录部分
- 示例： $(dir src/foo.c hacks) 返回值是 src/ ./ 
 
### notdir

`$(notdir <names...>)`
- 名称：取文件函数------notdir。
- 功能：从文件名序列 <names> 中取出非目录部分。非目录部分是指最后一个反斜杠（ / ） 之后的部分。
- 返回：返回文件名序列 <names> 的非目录部分。
- 示例: $(notdir src/foo.c hacks) 返回值是 foo.c hacks 。

### suffix

`$(suffix <names...>)`
- 名称：取后缀函数
- 功能：从文件名序列 <names> 中取出各个文件名的后缀。
- 返回：返回文件名序列 <names> 的后缀序列，如果文件没有后缀，则返回空字串。
- 示例： $(suffix src/foo.c src-1.0/bar.c hacks) 返回值是 .c .c。

### basename

`$(basename <names...>)`
- 名称：取前缀函数
- 功能：从文件名序列 <names> 中取出各个文件名的前缀部分。
- 返回：返回文件名序列 <names> 的前缀序列，如果文件没有前缀，则返回空字串。
- 示例： $(addprefix src/,foo bar) 返回值是 src/foo src/bar

### addsuffix

`$(addsuffix <suffix>,<names...>)`
- 名称：加后缀函数
- 功能：把后缀 <suffix> 加到 <names> 中的每个单词后面。
- 返回：返回加过后缀的文件名序列。

### addprefix

`$(addprefix <prefix>,<names...>)`
- 名称：加前缀函数
- 功能：把前缀 <prefix> 加到 <names> 中的每个单词前面。
- 返回：返回加过前缀的文件名序列。

### join

`$(join <list1>,<list2>)`
- 名称：连接函数
- 功能：把 <list2> 中的单词**对应**地加到 <list1> 的单词后面。如果 <list1> 的 单词个数要比 <list2> 的多，那么， <list1> 中的多出来的单词将保持原样。如果 <list2> 的单词个数要比 <list1> 多，那么， <list2> 多出来的单词将被复制到 <list1> 中。
- 返回：返回连接过后的字符串
- 示例： $(join aaa bbb , 111 222 333) 返回值是 aaa111 bbb222 333。

## foreach 函数

foreach 函数用来做循环用的  
语法是：
`$(foreach <var>,<list>,<text>)`

把参数 <list> 中的单词逐一取出放到参数 <var> 所指定的变量中， 然后再执行 <text> 所包含的表达式。每一次 <text> 会返回一个字符串，循环过程中， <text> 的所返回的每个字符串会以空格分隔，最后当整个循环结束时， <text> 所返回的每个字符串所组成的整个字符串（以空格分隔）将会是foreach函数的返回值。

## if 函数

if函数的语法是：  
`$(if <condition>,<then-part>)`
或是  
`$(if <condition>,<then-part>,<else-part>)`

<condition>参数是if的表达式，如果其返回的为非空字符串，那么这个表达式就相当于返回真，于是， <then-part> 会被计算，否则 <else-part> 会被计算。

## call函数

call函数是唯一一个可以用来创建新的参数化的函数。

`$(call <expression>,<parm1>,<parm2>,...,<parmn>)`

<expression> 参数中的变量，如 $(1) 、 $(2) 等，会 被参数 <parm1> 、 <parm2> 、 <parm3> 依次取代。而 <expression> 的返回值就是 call 函数的返回值。

注：在向 call 函数传递参数时要尤其注意空格的使用。第2个及其之后的参数中的空格会被保留  

## origin函数

origin函数是告诉你变量是哪里来的？  
其语法是：
`$(origin <variable>)`

注意， <variable> 是变量的名字，不应该是引用

Origin函数会以其返回值来告诉你这个变量的"出生情况"，
- undefined:    没有定义过
- default:      默认的定义
- environment:  环境变量，且 -e 参数没有被打开
- file:         被定义在Makefile中
- command line: 被命令行定义
- override:     被override指示符重新定义
- automatic:    命令运行中的自动化变量

## shell函数

shell 函数的参数应该就是操作系统Shell的命令。  
它和反引号"`"是相同的功能，把执行操作系统命令后的输出作为函数返回。  

注意，这个函数会新生成一个Shell程序来执行命令 

## 控制make的函数

make 提供了一些函数来控制 make 的运行。检测一些运行 Makefile 时的运行时信息，并且根据这些信息来决定，make 继续执行，还是停止。 
`$(error <text ...>)`  
产生一个致命的错误， <text ...>是错误信息。  
注意，error 函数不会在一被使用就会产生错误信息  

`$(warning <text ...>)`  
类 erro r函数，只是它并不会让 make 退出，只是输出一段警告信息  

# 运行

## 退出码

- 成功 0
- 错误 1
- 使用 -q 选项且部分目标不需要更新 2

## 指定 Makefile

使用 make 的 `-f` 或是 `--file` 参数（ --makefile 参数也行）给make 命令指定一个特殊名字的 Makefile  

## 指定目标

有一个 make 的环境变量叫 MAKECMDGOALS，这个变量中会存放所指定的终极目标的列表，如果在命令行上，没有指定目标，那么，这个变量是空值。

```
sources = foo.c bar.c
ifneq ( $(MAKECMDGOALS),clean)
    include $(sources:.c=.d)
endif
```

只要我们输入的命令不是"make clean"，那么makefile会自动包含"foo.d" 和"bar.d"这两个makefile。

- all:这个伪目标是所有目标的目标，其功能一般是编译所有的目标。
- clean:这个伪目标功能是删除所有被 make 创建的文件。
- install:这个伪目标功能是安装已编译好的程序，其实就是把目标执行文件拷贝到指定的目标中去。
- print:这个伪目标的功能是例出改变过的源文件。
- tar:这个伪目标功能是把源程序打包备份。也就是一个 tar 文件。
- dist:这个伪目标功能是创建一个压缩文件，一般是把 tar 文件压成 Z 文件或是 gz 文件。
- TAGS:这个伪目标功能是更新所有的目标，以备完整地重编译使用。
- check 和 test:这两个伪目标一般用来测试 makefile 的流程。

## 检查规则

### `-n, --just-print, --dry-run, --recon`
不执行参数，这些参数只是打印命令，不管目标是否更新，把规则和连带规则下的命令打印出来，但不执行，这些参数对于我们调 makefile 很有用处。

### `-t, --touch`
 把目标文件的时间更新，但不更改目标文件。 即 make 假装编译目标，但不是真正的编译目标，只是把目标变成已编译过的状态。

### `-q, --question`
找目标的意思。 如果目标存在，那么其什么也不会输出，当然也不会执行编译，如果目标不存在，其会打印出一条出错信息。

### `-W <file>, --what-if=<file>, --assume-new=<file>, --new-file=<file>`

需要指定一个文件。一般是是源文件（或依赖文件），Make 会根据规则推导来运行依赖于这个文件的命令，一般来说，可以和 "-n" 参数一同使用，来查看这个依赖文件所发生的规则命令。

## make 的参数

### `-b, -m`
忽略和其它版本make的兼容性。

### `-B, --always-make`

认为所有的目标都需要更新（重编译）。

### `-C <dir>, --directory=<dir>`

指定读取makefile的目录  
如果有多个"-C"参数，make 的解释是后面的路径以前面的作为相对路径，并以最后的目录作为被指定目录。  
如："make -C ~hchen/test -C prog" 等价于 "make -C ~hchen/test/prog"。

### `-debug[=<options>]`
输出make的调试信息
- a: 也就是all，输出所有的调试信息。
- b: 也就是basic，只输出简单的调试信息。即输出不需要重编译的目标。
- v: 也就是verbose，在b选项的级别之上。输出的信息包括哪个makefile被解析，不需要被重编译的依赖文件（或是依赖目标）等。
- i: 也就是implicit，输出所以的隐含规则。
- j: 也就是jobs，输出执行规则中命令的详细信息，如命令的PID、返回码等。
- m: 也就是 makefile，输出 make 读取 makefile，更新 makefile，执行 makefile 的信息
- 
### 参数选项

- `-d` 相当于"--debug=a"。
- `-e`, `--environment-overrides`: 指明环境变量的值覆盖 makefile 中定义的变量的值。
- `-f=<file>, --file=<file>, --makefile=<file>`: 指定需要执行的 makefile。
- `-h, --help`: 显示帮助信息。
- `-i , --ignore-errors`: 在执行时忽略所有的错误。
- `-I <dir>, --include-dir=<dir>`: 指定一个被包含 makefile 的搜索目标。可以使用多个"-I"参数来指定多个目录。
- `-j [<jobsnum>], --jobs [=<jobsnum>]`: 指同时运行命令的个数。如果没有这个参数，make 运行命令时能运行多少就运行多少。如果有一个以上的"-j"参数，那么仅最后一个"-j"才是有效的。
- `-k, --keep-going`: 出错也不停止运行。如果生成一个目标失败了，那么依赖于其上的目标就不会被执行了。
- `-l <load>, --load-average[=<load>], -max-load[=<load>]`: 指定make运行命令的负载。
- `-n, --just-print, --dry-run, --recon`: 仅输出执行过程中的命令序列，但并不执行。
- `-o <file>, --old-file=<file>, --assume-old=<file>`: 不重新生成的指定的<file>，即使这个目标的依赖文件新于它。
- `-p, --print-data-base`: 输出 makefile 中的所有数据，包括所有的规则和变量。
- ` -qp`命令：只是想输出信息而不想执行makefile  
- `--p --f /dev/null`：查看执行 makefile 前的预设变量和规则
- `-q, --question`: 不运行命令，也不输出。仅仅是检查所指定的目标是否需要更新。如果是0则说明要更新，如果是2则说 明有错误发生。
- `-r, --no-builtin-rules`: 禁止 make 使用任何隐含规则。
- `-R, --no-builtin-variabes`: 禁止 make 使用任何作用于变量上的隐含规则。
- `-s, --silent, --quiet`: 在命令运行时不输出命令的输出。
- `-S, --no-keep-going, --stop`: 取消"-k"选项的作用。
- `-t, --touch`: 只是把目标的修改日期变成最新的，也就是阻止生成目标的命令运行。
- `-v, --version`: 输出 make 程序的版本、版权等关于 make 的信息。
- `-w, --print-directory`: 输出运行 makefile 之前和之后的信息。这个参数对于跟踪嵌套式调用 make 时很有用。
- `--no-print-directory`: 禁止"-w"选项。
- `-W <file>, --what-if=<file>, --new-file=<file>, --assume-file=<file>`: 假定目标<file>需要更新，如果和"-n"选项使用，那么这个参数会输出该目标更新时的运行动作。如果没有"-n"那么使得<file>的修改时间为当前时间。
- `--warn-undefined-variables`: 只要 make 发现有未定义的变量，那么就输出警告信息。

# 隐含规则

"隐含规则"就是一种惯例，make 会按照这种"惯例"心照不喧地来运行，那怕 Makefile中没有书写这样的规则。  

1. 编译 C 程序的隐含规则。

`<n>.o` 的目标的依赖目标会自动推导为 `<n>.c`   
生成命令是 `$(CC) –c $(CPPFLAGS) $(CFLAGS)`

2. 编译 C++ 程序的隐含规则。

`<n>.o` 的目标的依赖目标会自动推导为 `<n>.cc` 或是 `<n>.C`   
生成命令是 `$(CXX) –c $(CPPFLAGS) $(CXXFLAGS)` 。

3. 汇编和汇编预处理的隐含规则。

`<n>.o` 的目标的依赖目标会自动推导为 `<n>.s `  
默认使用编译器 as 生成命令是： `$ (AS) $(ASFLAGS) `  
 `<n>.s`的目标的依赖目标会自动推导为 `<n>.S` ， 默认使用 C 预编译器 cpp ，并且其生成命令是：` $(AS) $(ASFLAGS`) 。

4. 链接 Object 文件的隐含规则。

`<n>` 目标依赖于 `<n>.o`，通过运行C的编译器来运行链接程序生成（一般是 ld ）  
生成命令是：` $(CC) $(LDFLAGS) <n>.o $(LOADLIBES) $(LDLIBS)`  
这个规则对于只有一个源文件的工程有效，同时也对多个 Object 文件（由不同的源文件生成）的也有效。

5. 从 C 程序、Yacc 文件或 Lex 文件创建 Lint 库的隐含规则。

`<n>.ln` （lint生成的文件）的依赖文件被自动推导为 n.c  
其生成命令是： `$(LINT) $(LINTFALGS) $(CPPFLAGS) -i `。  
对于 `<n>.y` 和` <n>.l` 也是同样的规则。

## 命令变量

- AR : 函数库打包程序。默认命令是 ar
- AS : 汇编语言编译程序。默认命令是 as
- CC : C语言编译程序。默认命令是 cc
- CXX : C++语言编译程序。默认命令是 g++
- CO : 从 RCS文件中扩展文件程序。默认命令是 co
- CPP : C程序的预处理器（输出是标准输出设备）。默认命令是 $(CC) –E
- FC : Fortran 和 Ratfor 的编译器和预处理程序。默认命令是 f77
- GET : 从SCCS文件中扩展文件的程序。默认命令是 get
- LEX : Lex方法分析器程序（针对于C或Ratfor）。默认命令是 lex
- PC : Pascal语言编译程序。默认命令是 pc
- YACC : Yacc文法分析器（针对于C程序）。默认命令是 yacc
- YACCR : Yacc文法分析器（针对于Ratfor程序）。默认命令是 yacc –r
- MAKEINFO : 转换Texinfo源文件（.texi）到Info文件程序。默认命令是 makeinfo
- TEX : 从TeX源文件创建TeX DVI文件的程序。默认命令是 tex
- TEXI2DVI : 从Texinfo源文件创建军TeX DVI 文件的程序。默认命令是 texi2dvi
- WEAVE : 转换Web到TeX的程序。默认命令是 weave
- CWEAVE : 转换C Web 到 TeX的程序。默认命令是 cweave
- TANGLE : 转换Web到Pascal语言的程序。默认命令是 tangle
- CTANGLE : 转换C Web 到 C。默认命令是 ctangle
- RM : 删除文件命令。默认命令是 rm –f

## 命令参数的变量

- ARFLAGS : 函数库打包程序 AR 命令的参数。默认值是 rv
- ASFLAGS : 汇编语言编译器参数。（当明显地调用 .s 或 .S 文件时）
- **CFLAGS : C语言编译器参数。**
- **CXXFLAGS : C++语言编译器参数。**
- COFLAGS : RCS命令参数。
- CPPFLAGS : C预处理器参数。（ C 和 Fortran 编译器也会用到）。
- FFLAGS : Fortran语言编译器参数。
- GFLAGS : SCCS "get"程序参数。
- LDFLAGS : 链接器参数。（如： ld ）
- LFLAGS : Lex文法分析器参数。
- PFLAGS : Pascal语言编译器参数。
- RFLAGS : Ratfor 程序的Fortran 编译器参数。
- YFLAGS : Yacc文法分析器参数。

# 自动化变量

自动化变量，就是这种变量会把模式中所定义的一系列的文件自动地挨个取出，直至所有的符合模式的文件都取完了。

自动化变量只应出现在规则的命令中。

- `$@ `: 表示规则中的目标文件集。在模式规则中，如果有多个目标  `$@` 就是匹配于目标中模式定义的集合。
- `$%` : 仅当目标是函数库文件中，表示规则中的目标成员名。例如，如果一个目标是 foo.a(bar.o) ， 那么， `$%` 就是 bar.o ， `$@` 就是 foo.a 。如果目标不是函数库文件其值为空。
- `$<` : 依赖目标中的第一个目标名字。如果依赖目标是以模式（即 %）定义的，那么 `$<` 将是符合模式的一系列的文件集。注意，其是一个一个取出来的。
- `$?` : 所有比目标新的依赖目标的集合。以空格分隔。
- `$^` : 所有的依赖目标的集合。以空格分隔。如果在依赖目标中有多个重复的，那么这个变量会去除重复的依赖目标，只保留一份。
- `$+` : 这个变量很像 `$^` ，也是所有依赖目标的集合。只是它不去除重复的依赖目标。
- `$*` : 这个变量表示目标模式中 % 及其之前的部分。如果目标是 dir/a.foo.b ，并且目标的模式是 a.%.b ，那么， $* 的值就是 dir/foo 。如果目标中没有模式的定义，那么 $* 也就不能被推导出，但是，如果目标文件的 后缀是make所识别的，那么 $* 就是除了后缀的那一部分。例如：如果目标是 foo.c ，因为 .c 是make所能识别的后缀名，所以， $* 的值就是 foo 。
  
这七个自动化变量还可以取得文件的目录名或是在当前目录下的符合模式的文件名，只需要搭配上 D 或 F 字样， 使用函数 dir 或 notdir 同样可以做到。

## 变量分别加上 D 或是 F 的含义

- $(@D): 表示 $@ 的目录部分（不以斜杠作为结尾）
  如果 $@ 值是 dir/foo.o ，那么 $(@D) 就是 dir ，而如果 $@ 中没有包含斜杠的话，其值就是 . （当前目录）。
- $(@F): 表示 $@ 的文件部分  
  如果 $@ 值是 dir/foo.o ，那么 $(@F) 就是 foo.o ， $(@F) 相当于函数 $(notdir $@) 。
- $(*D), $(*F): 取文件的目录部分和文件部分。
   $(*D) 返回 dir ， 而 $(*F) 返回 foo
- $(%D), $(%F): 分别表示了函数包文件成员的目录部分和文件部分。
- $(<D), $(<F): 分别表示依赖文件的目录部分和文件部分。
- $(^D), $(^F): 分别表示所有依赖文件的目录部分和文件部分。（无相同的）
- $(+D), $(+F): 分别表示所有依赖文件的目录部分和文件部分。（可以有相同的）
- $(?D), $(?F): 分别表示被更新的依赖文件的目录部分和文件部分。

# 更新函数库文件

函数库文件也就是对Object文件（程序编译的中间文件）的打包文件。  
在 Unix 下，一般是由命令 ar 来完成打包工作。

## 函数库文件的成员

一个函数库文件由多个文件组成。如下格式指定函数库文件及其组成:
```
archive(member)
```
例如：
```
foolib(hack.o) : hack.o
    ar cr foolib hack.o
```
多个 member 使用空格分开
```
foolib(hack.o kludge.o)
其等价于:
foolib(hack.o) foolib(kludge.o)
```
## 函数库成员的隐含规则

当 make 搜索一个目标的隐含规则时，如果这个目标是 a(m) 形式 的，其会把目标变成 (m) 。  
如果我们的成员是 %.o 的模式定义，且使用 make foo.a(bar.o) 的形式调用Makefile 时，隐含规则会去找 bar.o 的规则，如果没有定义 bar.o 的规则，那么内建隐含规则生效，make会去找 bar.c 文件来生成 bar.o 

## 函数库文件的后缀规则

使用"后缀规则"和"隐含规则"来生成函数库打包文件
```
.RECIPEPREFIX = >
.c.a:
>  $(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $*.o
>  $(AR) r $@ $*.o
>  $(RM) $*.o
```
其等效于：
```
.RECIPEPREFIX = >
(%.o) : %.c
>  $(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $*.o
>  $(AR) r $@ $*.o
>  $(RM) $*.o
```

# 例
## 例1： 
### 项目目录

```
<project>
├── build
|   ├─——— make.rule
|   |———— libs
|   |____ exes
|
└── src        
    ├── foo 
    |   |—— src
    |   |__ inc
    ├── bar 
    |   |—— src
    |   |__ inc
    |    
    └── hugo
        |__ main.c
```

### 文件内容
#### make.rule
```
.PHONY: all clean

MKDIR = mkdir
RM = rm
RMFLAGS = -fr

CC = gcc
AR = ar
ARFLAGS = crs

DIR_OBJS = objs
DIR_EXES = $(ROOT)/build/exes
DIR_DEPS = deps
DIR_LIBS = $(ROOT)/build/libs
DIRS = $(DIR_DEPS) $(DIR_OBJS) $(DIR_EXES) $(DIR_LIBS)
RMS = $(DIR_OBJS) $(DIR_DEPS)

ifneq ($(EXE), "")
EXE := $(addprefix $(DIR_EXES)/, $(EXE))
RMS += $(EXE)
endif

ifneq ($(LIB), "")
LIB := $(addprefix $(DIR_LIBS)/, $(LIB))
RMS += $(LIB)
endif

SRCS = $(wildcard *.c)
OBJS = $(SRCS:.c=.o)
OBJS := $(addprefix $(DIR_OBJS)/, $(OBJS))
DEPS = $(SRCS:.c=.dep)
DEPS := $(addprefix $(DIR_DEPS)/, $(DEPS))

ifneq ($(EXE), "")
all: $(EXE)
endif

ifneq ($(LIB), "")
all: $(LIB)
endif

ifneq ($(MAKECMDGOALS), clean)
include $(DEPS)
endif

ifneq ($(INC_DIRS),"")
INC_DIRS := $(strip $(INC_DIRS))
INC_DIRS := $(addprefix -I, $(INC_DIRS))
endif

ifneq ($(LINK_LIBS),"")
LINK_LIBS := $(strip $(LINK_LIBS))
LINK_LIBS := $(addprefix -l, $(LINK_LIBS))
endif

$(DIRS):
	$(MKDIR) $@

$(EXE): $(DIR_EXES) $(OBJS)
	$(CC) -L$(DIR_LIBS) -o $@ $(filter %.o, $^) $(LINK_LIBS)

$(LIB): $(DIR_LIBS) $(OBJS)
	$(AR) $(ARFLAGS) $@ $(filter %.o, $^)

$(DIR_OBJS)/%.o: $(DIR_OBJS) %.c
	$(CC) $(INC_DIRS) -o $@ -c $(filter %.c, $^)

$(DIR_DEPS)/%.dep: $(DIR_DEPS) %.c
	@set -e; \
	echo "Making $@..."; \
	$(RM) $(RMFLAGS) $@.tmp; \
	$(CC) $(INC_DIRS) -E -MM $(filter %.c, $^) > $@.tmp; \
	sed 's,\(.*\)\.o[ :]*,objs/\1.o $@: ,g' < $@.tmp > $@; \
	$(RM) $(RMFLAGS) $@.tmp

clean:
	$(RM) $(RMFLAGS) $(RMS)
```
#### foo
```
EXE =

LIB = libbar.a

INC_DIRS = $(ROOT)/source/bar/inc

LINK_LIBS =

include $(ROOT)/build/make.rule

```
#### main
```
EXE = huge.exe

LIB =

INC_DIRS = $(ROOT)/source/foo/inc \
	$(ROOT)/source/bar/inc

LINK_LIBS = foo bar

include $(ROOT)/build/make.rule

```
## 例2：
### 项目目录
```
<project>
├── Makefile
├── build
└── src
    ├── hello.c
    ├── hello.h
    └── main.c
```
### Makefile 文件内容
```
SRC_DIR = ./src
BUILD_DIR = ./build
TARGET = $(BUILD_DIR)/world.out

CC = cc
CFLAGS = -Wall

# ./src/*.c
SRCS = $(shell find $(SRC_DIR) -name '*.c')
# ./src/*.c => ./build/*.o
OBJS = $(patsubst $(SRC_DIR)/%.c,$(BUILD_DIR)/%.o,$(SRCS))
# ./src/*.c => ./build/*.d
DEPS = $(patsubst $(SRC_DIR)/%.c,$(BUILD_DIR)/%.d,$(SRCS))

# 默认目标:
all: $(TARGET)

# build/xyz.d 的规则由 src/xyz.c 生成:
$(BUILD_DIR)/%.d: $(SRC_DIR)/%.c
	@mkdir -p $(dir $@); \
	rm -f $@; \
	$(CC) -MM $< >$@.tmp; \
	sed 's,\($*\)\.o[ :]*,$(BUILD_DIR)/\1.o $@ : ,g' < $@.tmp > $@; \
	rm -f $@.tmp

# build/xyz.o 的规则由 src/xyz.c 生成:
$(BUILD_DIR)/%.o: $(SRC_DIR)/%.c
	@mkdir -p $(dir $@)
	$(CC) $(CFLAGS) -c -o $@ $<

# 链接:
$(TARGET): $(OBJS)
	@echo "buiding $@..."
	@mkdir -p $(dir $@)
	$(CC) -o $(TARGET) $(OBJS)

# 清理 build 目录:
clean:
	@echo "clean..."
	rm -rf $(BUILD_DIR)

# 引入所有 .d 文件:
-include $(DEPS)

```

