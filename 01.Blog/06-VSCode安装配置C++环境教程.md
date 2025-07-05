# 前言
IDE——集成开发环境，用于提供程序开发环境，集成了代码编写、分析、编译和调试等一体化的的套件。如C++的Visual Studio、Java的IDEA和Python的PyCharm等。IDE部署配置简单、功能全面，无论是作为新手入门还是项目开发都是更好的选择。  
但如何如果你喜欢折腾,焦虑存储空间，需要多语言开发。VS Code这一个宇宙第一IDE(自己搭建后)一定要尝试一下。支持C/C++、Java、Python...等你所需要的绝大部分语言，攥写文章也有不错的体验，总之，基于其丰富的插件生态，它能满足你的一切。  
这里重点吐槽一下，可能是为了不和Visual Studio重复，VS Code配置C/C++环境麻烦了不少。这里对此进行记录，~~~作为一个遇事不决重装系统的人来说，都配置麻了~~~

# C/C++编译器安装
由于VS Code本体不具备C/C++编译器，首先需要对此进行安装来支持代码的编译和运行。具体可选择minGW、Clang、MSYS2等，主流是MinGW64或MSYS2.可以自行了解其区别。这里选择安装MSYS2后下载minGW64。  
注：可以直接在[官方网站](https://sourceforge.net/projects/mingw-w64/)下载MinGW64安装包解压后在配置环境变量，这里主要是博主使用的其它软件必须依赖MSYS2。
## 1.下载安装msys2
从[MSYS2官方网站](https://www.msys2.org/)下载软件。
![msys2setup](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/msys2setup.png)
下载完成后点击安装即可，需要注意的是自定义安装路径时不要有空格、中文等非ASCII字符。
## 2.安装MinGW64
安装完成后一般默认打开"MSYS2 MSYS",如何没有则在开始菜单中自行查找打开。  
首先输入运行`pacman -Syu`更新安装包， 需要确认时有[Y/N]提示时输入Y,否则直接回车。  
 ![msys01](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/msys01.png)
上述命令执行后重新打开"MSYS2 MSYS"执行`pacman -Su`,操作同理。
![msys02](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/msys02.png)
之后安装 mingw-w64 GCC。执行`pacman -S --needed base-devel mingw-w64-x86_64-toolchain`。
![msys03](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/msys03.png)
## 3.配置环境变量
打开MSYS2的安装路径，可以看到mingw64文件夹，记录其下bin目录路径，如`C:\MSYS2\mingw64\bin`。配置环境变量，具体在高级系统设置中环境变量Path中。  
![msys04](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/msys04.png)
配置完成后打开CMD输入`gcc --version`终端存在信息输出则配置成功。  
# VS Code软件下载安装
[VS Code官网](https://code.visualstudio.com/Download)
软件下载安装过程不在赘述。安装完成后在插件商店安装相关插件:Chinese、C/C++。  
![vscodeset](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/vscodeset.png)
# VS Code C++环境配置
直接将他人配置好的.vscode文件导入项目修改minGW64路径，包括c_cpp_properties.json、lanunch.json、settings.json和tasks.json。  这样就可以直接使用了，否则手动进行配置。
## 1.配置C/C++插件
 ctrl+shift+p打开设置C/C++:编辑配置  
 ![vscodeset01](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/vscodeset01.png)
如果要配置多个选项在此添加。
![vscodeset05](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/vscodeset05.png) 
编译器路径选择mingw64下的gcc.exe或g++.exe。更推荐g++.exe
 ![vscodeset03](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/vscodeset03.png)
 IntelliSense模式选择gcc即可。
![vscodeset02](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/vscodeset02.png)

最后指定C/C++的语言标准，推荐C11、C++11。一般都向下兼容。
![vscodeset04](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/vscodeset04.png)
## 2.程序编译运行
点击文件，打开一个文件夹作为工作区。
![cset01](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/cset01.png)
新建源文件编译一个简单代码、运行。
![cset02](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/cset02.png)
点击编译器，注C++必须要g++.exe,这里的选项由之前的配置决定。
![cset03](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/cset03.png)  
之后一般会直接运行成功，并生成.vscode目录和tasks.json文件。对tasks.json文件内容按照mingw路径进行修改。
![cset04](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/cset04.png)
如果需要使用断点调试则配置lanuch.json文件，步骤如下：
创建lanuch.json文件，选择GDB。
![cset05](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/cset05.png)
添加配置选择gdb启动。
![cset06](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/cset06.png)
对文件内容中的程序名和gdb.exe路径进行修改。
![cset07](https://noxrynbed.oss-cn-chengdu.aliyuncs.com/img/cset07.png)