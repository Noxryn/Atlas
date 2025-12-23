# 前言
规范的注释是一个良好的编程习惯。
Doxygen可以直接将注释提取为程序文档，便于开发人员使用
本文主要介绍了Doxygen的使用方法，更多细节请阅读[官方文档](https://www.doxygen.nl/)

---
# 一、Doxygen是什么？
doxygen是一个文档生成工具，用于为源码生成文档，我们可以在代码中按照doxygen要求的语法编写代码注释，然后使用doxygen将程序中的注释提取出来生成一个文档
我们一般都把注释写在代码中，所以使用doxygen生成的文档也可以非常方便地更新。Doxygen还会在文档中引用源码文件，故我们也可以在文档中方便地查看源码
# 二、Doxygen的安装
## 1.windows
下载软件.软件安装过程基本无脑下一步
## 2.Linux
### (1).使用命令安装
```
sudo apt-get install doxygen
sudo apt-get install doxygen-gui
```
### (2).使用源码安装
下载压缩包：如doxyen-1.11.0.src.tar.gz
```
tar zxvf doxyen-1.11.0.src.tar.gz
cd  doxyen-1.11.0
./configure
make
sudo make install
```
### (3).安装Graphviz和HTML Help Workshop(可选)
Doxygen 使用HTML Help Workshop可以生成 CHM 格式的文档
Graphviz在Doxygen用于自动生成类图的工具。
官网：[Graphviz](https://graphviz.org/)和[HTML Help Workshop](https://www.helpandmanual.com/downloads_mscomp.html)
注：记住安装路径，配置到Doxygen中
# 三、DoxygenHTML HELP的使用
## 1.配置Wizard
![doxy-1](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/doxy-1.png)
注：工作目录和源文件目录一般保持一致
![doxy-2](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/doxy-2.png)
![doxy-3](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/doxy-3.png)
![doxy-4](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/doxy-4.png)
## 2.配置Expert
![doxy-5](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/doxy-5.png)
![doxy-6](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/doxy-6.png)
## 3.运行
![doxy-7](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/doxy-7.png)
# 四、Doxygen语法
## 1.特殊命令
| 命令           | 字段名                                     | 语法                                                      |
| -------------- | ------------------------------------------ | --------------------------------------------------------- |
| @file          | 文件名                                     | file [< name >]                                           |
| @brief         | 简介                                       | brief { brief description }                               |
| @author        | 作者                                       | author { list of authors }                                |
| @mainpage      | 主页信息                                   | mainpage [(title)]                                        |
| @date          | 年-月-日                                   | date { date description }                                 |
| @author        | 版本号                                     | version { version number }                                |
| @copyright     | 版权                                       | copyright { copyright description }                       |
| @param         | 参数                                       | param [(dir)] < parameter-name> { parameter description } |
| @return        | 返回                                       | return { description of the return value }<br>            |
| @retval        | 返回值                                     | retval { description }                                    |
| @bug           | 漏洞                                       | bug { bug description }                                   |
| @details<br>   | 细节                                       | details { detailed description }                          |
| @pre           | 前提条件                                   | pre { description of the precondition }<br>               |
| @post          | 说明代码项之后的使用条件                   | { description of the postcondition }                      |
| @see           | 参考                                       | see { references }                                        |
| @link          | 连接`(与@see类库，{@link www.google.com})` | link < link-object>                                       |
| @throw         | 异常描述                                   | throw < exception-object> { exception description }       |
| @exception<br> | 对一个异常对象进行注释                     | {exception description}                                   |
| @todo          | 待处理                                     | todo { paragraph describing what is to be done }          |
| @warning       | 警告信息                                   | warning { warning message }                               |
| @example       | 弃用说明。可用于描述替代方案，预期寿命等   | deprecated { description }                                |
| @deprecated    | 弃用说明。可用于描述替代方案，预期寿命等   | deprecated { description }                                |
| @code          | 在注释中开始说明一段代码，直到@endcode命令 |                                                           |
| @endcode       | 注释中代码段的结束                         |                                                           |
| @since         | 通常用来说明从什么版本、时间写此部分代码。 | {text}                                                    |
| @attention     | 注意                                       |                                                           |
## 2.注释风格
### (1).文件注释
注：位于文件开头
```
/**
 * @file 文件名
 * @brief 简介
 * @details 细节
 * @author 作者
 * @version 版本号
 * @date 年-月-日
 * @copyright 版权
 */
```
### (2).函数注释
```
 /**
  * @brief 函数描述
  * @param 参数描述
  * @return 返回描述
  * @retval 返回值描述
  */
```
### (3).常量/变量注释
一般常量/变量可以有两种形式：
- 常量/变量上一行注释
- 常量/变量后注释
```
/// 定义一个变量
int a;

int a; ///< 定义一个变量
```


​