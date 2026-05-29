# 前言

Windows 环境下开发 C++ 时引入第三方库是一个很麻烦的事情，编译部分库时甚至可能遇到花费几天却白费功夫的情况。  

vcpkg 作为一个开源的 C++ 库包管理工具，可以轻松地安装和管理各种C++库，而无需手动下载和配置。

# 安装

官网地址：[vcpkg](https://vcpkg.io/en/index.html)
1. 创建存放目录
2. 管理员权限打开 PowerShell
3. 下载 vcpkg
    ```
    git clone https://github.com/Microsoft/vcpkg.git
    ```
4. 进入vcpkg 目录
    ```
    cd vcpkg
    ```
5. 运行批处理文件
    ```
    .\bootstrap-vcpkg.bat
    ```
6. 添加环境变量
7. 更新
    ```
    vcpkg update
    ```

# 命令

| 命令                                    | 作用                       |
| --------------------------------------- | -------------------------- |
| vcpkg search [name]                     | 搜索第三方库               |
| vcpkg install [name]                    | 安装包                     |
| vcpkg remove [name]                     | 卸载包                     |
| vcpkg remove --outdated                 | 卸载过期包                 |
| vcpkg list                              | 列出安装的包               |
| vcpkg update                            | 更新包                     |
| vcpkg upgrade                           | 重新生成过期包             |
| vcpkg hash <file> [alg]                 | 使用算法对文件执行哈希操作 |
| vcpkg integrate install                 | 全局集成                   |
| vcpkg integrate remove                  | 移除全局集成               |
| vcpkg integrate [project]               | 项目集成                   |
| vcpkg export [name] [opt]               | 导出包                     |
| vcpkg import [name]                     | 导入包                     |
| vcpkg create [name] <url> [archivename] | 创建新程序包               |
| vcpkg cache                             | 列出缓存的已编译包         |
| vcpkg version 显示版本信息              |
| vcpkg contact --survey                  | 显示联系方式               |
| vcpkg help triplet                      | 查看支持的平台             |

注：vcpkg支持导出参数  
| 参数                                | 作用                                                       |
| ----------------------------------- | ---------------------------------------------------------- |
| -raw                                | 不打包                                                     |
| -nuget                              | nuget 包形式                                               |
| -ifw                                | 于IFW的安装程序                                            |
| -zip                                | zip 压缩包形式                                             |
| -7zip                               | 7z 压缩包形式                                              |
| --dry-run                           | 仅显示哪些库将会被导出，而不执行实际的导出命令             |
| --output=…                          | 指定输出文件的名称                                         |
| --output-dir=…                      | 指定文件的输出目录                                         |
| --nuget-id=…                        | 指定导出的NuGet包的ID。配合--nuget使用                     |
| --nuget-description=…               | 为导出的NuGet包指定一个描述                                |
| --nuget-version=…                   | 指定导出的NuGet包的版本                                    |
| --ifw-repository-url=...            | 指定在线安装程序的远程存储库URL                            |
| --ifw-packages-directory-path=...   | 指定重新打包的软件包的临时目录路径                         |
| --ifw-repository-directory-path=... | 指定导出的版本库的目录路径                                 |
| --x-all-installed                   | 导出所有已安装的库                                         |
| --x-chocolatey                      | 导出一个Chocolatey软件包，必须和--x-maintainer=...同时使用 |
| --x-maintainer=...                  | 为导出的Chocolatey软件包指定维护者                         |
| --x-version-suffix=...              | 指定为导出的Chocolatey包添加版本后缀                       |
| --prefab                            | 导出为Prefab格式                                           |
| --prefab-maven                      | 启用maven                                                  |
| --prefab-debug                      | 启用prefab调试功能                                         |
| --prefab-group-id=...               | GroupId根据maven规范唯一标识您的项目                       |
| --prefab-artifact-id=...            | Artifact Id是maven规范中的项目名称                         |
| --prefab-version=...                | 版本是根据maven规范的项目名称                              |
| --prefab-min-sdk=...                | Android支持的最低sdk版本                                   |
| --prefab-target-sdk=...             | Android目标sdk版本                                         |

# 集成

## 全局集成

```
vcpkg intergrate install
vcpkg integrate remove
```
## 工程集成

1. 生成配置文件
    ```
    vcpkg integrate <project>
    ```
    运行命令后会在 vcpkg 安装目录`<vcpkg_dir>`\scripts\buidsystems 下生成 nuget 配置文件

    注：`<vcpkg_dir>` 为 vcpkg 安装目录

2. 基本设置
打开 Visual Studio ，点击 **工具->NuGet包管理器->程序包管理器设置**,点击**程序包源**，增加源，选择上述配置文件所在目录

3. 工程配置，
选择设置的工程，打开**管理NuGet程序包**，选择设置好的源，安装vcpkg

## CMake 集成

```
-DCMAKE_TOOLCHAIN_FILE=<vcpkg_dir>/scripts/buildsystems/vcpkg.cmake"  
```
### 集成静态库

Vcpkg 默认编译链接的是动态库 

1. VS 中集成静态库
修改 vcxproj 工程文件，在 xml 段中增加
    ```
    <VcpkgTriplet>x86-windows-static</VcpkgTriplet>
    <VcpkgEnabled>true</VcpkgEnabled>
    ```

2. CMake 中集成静态库
    ```
    cmake .. -DCMAKE_TOOLCHAIN_FILE=<vcpkg_dir>/scripts/buildsystems/vcpkg.cmake -DVCPKG_TARGET_TRIPLET=x86-windows-static
    ```
