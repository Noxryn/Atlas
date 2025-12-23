# 前言

对常用 Markdown 语法的进行了归纳总结，仅供参考。  

Markdown 语法简单，只需记忆少量标记即可愉快使用，个人建议花少量时间了解语法后直接练习。 
      
---

# 1.Markdown是什么？

Markdown 是一种轻量级的标记语言，可用于在纯文本文档中添加格式化元素。

Markdown 由 John Gruber 于 2004 年创建，如今已成为世界上最受欢迎的标记语言之一。

基本特性：
- 专注于文字内容
- 纯文本，易读易写，可以方便地纳入版本控制
- 语法简单，能轻松在码字的同时做出美观大方的排版

注：  
尽管 Markdown语法被广泛支持，但
> 实际上每个 Markdown 应用程序都实现了稍有不同的 Markdown 语法  

对此只能在使用中自行注意

# 2.语法表 

| 元素     | 语法                        | 元素     | 语法                                        |
| -------- | --------------------------- | -------- | ------------------------------------------- |
| 标题     | # H1                        | 无序列表 | - First item                                |
| 粗体     | `** bold text **`           | 代码     | `` ` code ` ``                              |
| 斜体     | `*italicized text*`         | 分割号   | ---                                         |
| 引用块   | > blockquote                | 链接     | `[title](https:://www.example.com "title")` |
| 有序列表 | 1. First item               | 图片     | `![alt text](image.jpg)`                    |
|          | \|syntax\|Description\|     | 标题编号 | ### Heading {#custom-id}                    |
| 表格     | \|-------\|------------\|   | 定义列表 | term                                        |
|          | \|header\|    title      \| |          | :  definition                               |
| 代码块   | ``` <br>code<br>````        | 删除线   | ~~ text ~~                                  |
| 脚注     | hello world.`[^1]`          | 任务列表 | - [ ] title                                 |
|          | `[^1]:description`          |          | -[ x] title                                 |

注：
- 文章基于 Markdown 编辑器书写，即时渲染
- 代码语法为单引号，代码块语法为三重单引号
- 其它位置出现单引号皆为使用代码语法保持文章显示正确
- Markdown 支持 HTML 语法，原生不支持功能，可用 HTML 语言进行拓展

 [emoji表情简表](https://gist.github.com/rxaviers/7360908 "emoji表情简表")

## 特殊字符

```
版权 (©)      —   &copy;  
注册商标 (®)  —   &reg;  
欧元 (€)     —   &euro;  
左箭头 (←)   —   &larr;  
上箭头 (↑)   —   &uarr;  
右箭头 (→)   —   &rarr;  
下箭头 (↓)   —   &darr;   
度数 (°)     —   &#176;  
圆周率 (π)   —   &#960;  
```
---
