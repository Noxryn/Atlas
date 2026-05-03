# 前言
记录Git软件使用相关的流程、命令。   

---

# Git是什么？
Git 是一个用于管理源代码的分布式版本控制系统。 版本控制系统会在您修改文件时记录并保存更改，使用户可以随时恢复以前的工作版本。使用 Git可以轻松访问源代码的修改历史记录以及更改的人，同时防止旧版本的无意覆盖。

# Git下载安装
## 1.下载地址  
进入[git-scm网站](https://git-scm.com/downloads)
后下对应系统版本的安装包
![git-01](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/git-01.png)
## 2.注意事项
安装包下载后双击安装，安装向导进行配置。具体操作可参考博文[Git 详细安装教程](https://blog.csdn.net/mukes/article/details/115693833?fromshare=blogdetail&sharetype=blogdetail&sharerId=115693833&sharerefer=PC&sharesource=Noxryn&sharefrom=from_link)
# Git工作流程
![git-02](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/git-02.png)
# Git命令速成
## 1.软件初始配置
```
git config --global user.name <用户名> # 设置用户名
git config --global user.email <邮箱名> # 设置邮箱
git config --global core.excludesfile ~/.gitignore_global # 全局gitignore
```
## 2.本地库管理
```
git init # 初始本地库
git status # 查看本地库状态
git diff # 查看详细更新状态，尚未缓存的改动
git diff --cached # 查看已缓存的改动
git diff HEAD # 查看已缓存的与未缓存的所有改动
git diff --stat # 显示摘要而非整个diff
git add <文件名> # 添加暂存区
git rm --cached <文件名> # 删除暂存区文件
git mv <old text> <new text> # 移动或重命名一个文件、目录、软连接
git commit -m "<日志信息>" <文件名> # 提交本地库
```
## 3.查看历史记录
```
git reflog
git log [-options] # 详细信息
   # –oneline # 查看历史记录的简洁版本
   # –graph # 查看历史中什么时候出现了分支、合并
   # –reverse # 逆向显示所有日志
   # –author # 查找指定用户的提交日志
   # –since、–before、 --until、–after # 定筛选日期
   # –no-merges # 选项以隐藏合并提交
```
## 4.标签
```
git tag # 查看标签
git tag -a vx.x # 创建标签
git tag -a vx.x <哈希值> # 追加标签
git tag -a <tagname> -m "某某标签"  # 指定标签信息
git tag -s <tagname> -m "某某标签" # PGP签名标签
```
## 5.分支管理
```
git branch -v # 查看分支
git branch <分支名> # 创建分支
git branch -d  <分支名> # 删除分支
git checkout <分支名> # 切换分支
git checkout <提交记录> # 切换HEAD
git checkout -b <分支名>  # 创建并切换分支
git merge <要合并的分支>  # 合并分支,合并冲突解决后提交本地库时不加文件名
git rebase <分支名> # 将当前分支复制到指定分支
git cherry-pick <哈希值> <哈希值> # 将其它分支的指定提交记录复制到当前分支
```
## 6.版本穿梭
```
git reset --hard <版本号> //回退分支，取消已缓存的内容
git checkout HEAD^ //移动到当前位置的父节点位置
git branch -f <分支名> HEAD~num //移动分支到num前
```
## 7.远程仓库管理
```
git clone <远程链接> # 克隆仓库
ssh -keygen -t rsa -C 远仓ssh地址 # git生成ssh公钥
git remote -v # 查看远程库信息
git remote add <别名> <远程库地址> # 关联远程库
git remote remove <远程库别名> # 删除关联库
git remote rename <旧别名> <新别名> # 修改关联库名称
git remote set-url <别名> <新链接> # 重设远程库地址
git push <远程库别名> <分支名>  # 推送远程库
git pull <远程库别名> <分支名>  # 拉取远程库
git fetch <远程库别名>  # 拉取远程库最新数据，不自动合并
```
## 8.gitignore文件
gitignore文件用使Git忽略指定文件。  
```
# VS Code
.vscode/

# C/C++
*.d
*.slo
*.lo
*.o
*.obj
*.gch
*.pch
*.so
*.dylib
*.dll
*.mod  
*.smod
*.lai  
*.la  
*.a  
*.lib
*.exe  
*.out  
*.app
/x64/
/x86/

# Java

# zip
*.zip
*.tar.gz
*.rar

# obsidian
.obsidian
```

# 日志提交规范

| 类型     | 介绍         | 脚注            | 功能       |
| -------- | ------------ | --------------- | ---------- |
| feat     | 新增功能     | BREAKING CHANGE | 破坏性修改 |
| fix      | 修改bug      | Closes #`<num>` | 关闭issue  |
| to       | 发现bug      |                 |            |
| docs     | 文档更新     |                 |            |
| style    | 格式更新     |                 |            |
| refactor | 重构代码     |                 |            |
| perf     | 性能优化     |                 |            |
| test     | 测试用例     |                 |            |
| chore    | 其它修改     |                 |            |
| ci       | CI配置修改   |                 |            |
| build    | 构建系统变更 |                 |            |
| release  | 发布新版本   |                 |            |

