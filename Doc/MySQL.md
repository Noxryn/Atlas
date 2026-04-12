# 基础篇

## 概述

功能：数据持久化

DB:数据库    
DBMS:数据库管理系统    
SQL:结构化查询语言      
基础：InnoDB 引擎

### RDBMS vs NoSQL

#### RDBMS

- 二维表格
- 以行（row）和列(col)形式存储数据，行列 -> 表（table）-> 库（database）
- 表与表之间的数据记录存在关系，数据库建立在关系模型上

优势：  
- 复杂查询：便于在多表间进行复杂数据查询
- 事务支持 ：实现对安全性能高的数据访问

#### NoSQL

优势：性能高  
  
1. 键值对数据库（Redis）    
   - 基于 Key-Value 键值存储数据
   - 用于内存缓存
2. 文档型数据库（MongoDB）
   - 以文档（XML、JSON等）作为信息处理的基本单位
3. 搜索引擎数据库（Solr、ELasticsearch、Splunk）
   - 搜索引擎领域
   - 倒排索引
4. 列式数据库（HBase）
   - 数据按列存储
   - 降低系统 I/O
   - 适合分布式系统
   - 功能相对受限

5. 图形数据库（Neo4J、infoGrid）
   - 数据结构-图存储实体之间的关系
   - 以节点和边实现
  
### 设计规则

#### 数据对象

| 对象     | 描述                   |
| -------- | ---------------------- |
| 表       | 存储数据的逻辑单元     |
| 数据字典 | 存放数据库信息的表     |
| 约束     | 数据校验               |
| 视图     | 数据的逻辑显示         |
| 索引     | 提高查询性能           |
| 存储过程 | 完成一次完整的业务处理 |
| 存储函数 | 完成一次特定计算       |
| 触放器   | 事件监听器             |

#### 表

E-R(实体-联系)模型:实体集、属性、联系集
- 实体集:表
- 实体:行，记录
- 属性:列，字段

#### 关系

- **一对一**
- **一对多**
- **多对多**
  

## SQL  

### 分类

- DDL：数据定义语言（create\alter\drop\rename\truncate），不可回滚
- DML：数据操作语言（insert\delete\update\select） 设置`SET autocommit = FALSE` 后数据可回滚
- DCL：数据控制语言（commit\rollback\savepoint\grant\revoke）

### 规范

- 子句分行编写
- 命令以`;`结尾
- 关键字不能缩写分行
- 数据类型以单引号表示
- 列别名以双引号表示
- 单行注释（`#`）、多行注释（`/* */`）

### DDL

```
CREATE DATABASE [IF NOT EXISTS] <name> [CHARACTER SET 'utf8/gbk']; # 创建数据库
SHOW DATABASES # 显示数据库
SHOW CREATE DATABASE <name>; # 查看数据库结构
USE <database name>; # 切换数据库
SHOW TABLES [FROM database]; # 查看数据表
SELECT DATABASE() FROM DUAL; # 查看当前使用的数据库
ALTER <database> CHARACTER SET <'字符串'>; # 修改字符集
DROP DATABASE [IF EXISTS] <name>; # 删除数据库

CREATE TABLE [IF NOT EXISTS] <name(val type,...)>; # 创建表

CREATE TABLE <new name>
AS 
<SELECT 字段... FROM <old table>>;

DESC <table name>; # 查看表结构
SHOW CREATE TABLE <table name>;

SELECT * FROM <name>; # 查看表数据

ALTER  TABLE <name>   # 修改表
ADD <字段> <type> [FIRST/ AFTER <字段字段>]  # 添加新字段[最后/指定字段后]
MODIFY <字段> <VARCHAR(35) DEFAULT ''> # 修改字段
CHANGE <字段> <new_name type> # 重命名字段
DROP COLUMN <字段> # 删除字段
RENAME TO <new name> # 重命名表 等价于 RENAME TABLE <old> TO <new>; 

DROP TABLE [IF EXISTS] <name>； # 删除表 数据和表结构

TRUNCATE TABLE <name> # 清空表 保存表结构 不可回滚
DELETE FROM <name>    # 可以回滚
```

### TCL（事务）

- 原子性；成功 OR 回滚


`COMMIT` ：提交数据。不可回滚
`ROLLBACK`：回滚数据到最近一次 `COMMIT`

### 数据类型

| 类型       | 举例                                                                |
| ---------- | ------------------------------------------------------------------- |
| 整型       | TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT                           |
| 浮点       | FLOAT、DOUBLE(总位数M，小数位数D)、REAL(DOUBLE)                     |
| 定点数     | **DECIMAL(M,D)**,字节 M+2                                           |
| 位类型     | BIT                                                                 |
| 日期       | YEAR、TIME、DATE、DATETIME、TIMESTAMP                               |
| 文本字符串 | CHAR、VARCHAR、TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT                 |
| 枚举       | ENUM                                                                |
| 集合       | SET                                                                 |
| 二进制     | BINARY、VARBINARY、TINYBLOB、BLOB、MEDIUMBLOB、LONGBLOB             |
| JSON       | 对象、数组                                                          |
| 空间数据   | 单值：GEOMETRY、POINT、LINESTRING、POLYGON                          |
|            | 集合：MULTIPOINT、MULTILINESTRING、MULTIPOLYGON、GEOMETRYCOLLECTION |

| 类型      | 字节 | 取值                   | 无符号取值   | 用途               |
| --------- | ---- | ---------------------- | ------------ | ------------------ |
| TINYINT   | 1    | -128~127               | 0~255        | 枚举，取值小且固定 |
| SMALLINT  | 2    | -32768~32767           | 0~65535      | 小范围统计         |
| MEDIUMINT | 3    | -8288608~83888607      | 0~16777215   | 较大整数计算       |
| **INT**   | 8    | -2147483648~2147483647 | 0~4294967295 | 不考虑超限         |
| BIGINT    | 8    |                        |              | 巨量数据           |

~~显示宽带属性M，指定显示宽度，不足补零：INT(val) ZEROFILL~~

| 类型         | 名称     | 字节 | 格式                | 最小值                  | 最大值                  |
| ------------ | -------- | ---- | ------------------- | ----------------------- | ----------------------- |
| YEAR         | 年       | 1    | YYYY                | 1901                    | 2155                    |
| TIME         | 时间     | 3    | HH:MM:SS            | -838:59:59              | 838:59:59               |
| DATE         | 日期     | 3    | YYYY-MM-DD          | 1000-01-01              | 9999-12=03              |
| **DATETIME** | 日期时间 | 8    | YYYY-MM-DD HH:MM:SS | 10001-01-01 00:00:00    | 9999-12-31 23:59:59     |
| TIMESTAMP    | 日期时间 | 4    | YYYY-MM-DD HH:MM:SS | 1970-01-01 00:00:00 UTC | 2038-01-19 03:14:07 UTC |


| 类型             | 范围         | 字节          |
| ---------------- | ------------ | ------------- |
| CHAR(M)          | 0~255        | M             |
| **VARCHAR(M)**   | 0~65535      | M+1           |
| TINYTEXT         | 0~255        | L+2           |
| TEXT （not key） | 0~65535      | L+2           |
| MEDIUMTEXT       | 0~16777215   | L+2           |
| LONGTEXT         | 0~4294967295 | L+4           |
| ENUM             | 1~65535      | 1、2          |
| SET              | 0~64         | 1、2、3、4、8 |

| 关键字             | 含义       |
| ------------------ | ---------- |
| NULL               | 可以为空   |
| NOT NULL           | 不可为NULL |
| DEFAULT            | 默认值     |
| PRIMARY KEY        | 主键       |
| AUTO_INCREMENT     | 自动递增   |
| UNSIGNED           | 无符号     |
| CHARACTER SET name | 指定字符集 |

### 约束

完整性：数据的精确性和可靠性  
- 实体完整性：唯一记录
- 域完整性：范围确定
- 引用完整性：引用正确
- 用户自定义完整性

```
# 查看约束
SELECT * FROM information_schema.table_constraints 
```

#### not null (非空约束)

```
CREATE TABLE table(  # 添加约束
    id INT NOT NULL
)

ALTER TABLE table 
MODIFY 字段 类型 NOT NULL;

ALTER TABLE table  # 删除约束
MODIFY 字段 类型 NULL;
```

#### unique (唯一性约束)  

```
CREATE TABLE table(
    # 行级约束
    id INT UNIQUE # 添加约束
    # 表级约束
    CONSTRAINT 约束名 UNIQUE(字段,...)
)

# 删除唯一约束只能删除唯一索引，默认列名
ALTER TABLE table
DRORP INDEX 唯一索引；
```

**复合唯一性约束下数据可以部分一致**

#### primary key (主键约束)

- 主键唯一

```
CREATE TABLE table(  # 添加约束
    id INT NOT NULL PRIMARY KEY,
    
    PRIMARY KEY (字段,...) # 表级约束
)


ALTER TABLE table # 删除约束
DRORP PRIMARY KEY；
```

#### foreign key (外键约束)  

**更推荐在应用层面完成，外键影响插入速度**

外键约束不能跨引擎使用！ 

```
CREATE TABLE base(
    id INT PRIMARY KEY
);

CREATE TABLE son(
    son_id INT PRIMARY KEY,
    base_id INT,
    
    CONSTRAINT fk_son_base_id FOREIGN  KEY (base_id) REFERENCES base(id)  
)


# 删除约束
# 删除外键约束
ALTER TABLE <table>
DROP FOREIGN KEY <约束名>

SHOW INDEZ FROM <table>

# 删除外键约束对应索引
ALTER TABLE <table>
DROP INDEX <约束名>
```

##### 约束等级

- **Cascade**：父表更新，同步子表
- **SET NULL**：父表更新，子表为空
- **NO ACTION**：不允许父表更新
- **RESTRICT**：同 NO ACTION
- SET DEFAULT：父表更新，子表设为默认值
- `ON UPDATE CASCADE ON DELETE RESTRICT`

#### check (检查约束)

检查数据是否满足条件

```
CREATE TABLE base(
    id INT CHECK(条件)
```

#### default (默认约束)  

设置数据默认值

```
CREATE TABLE base(
    id INT DEFAULT <val>
)

ALTER TABLE table  # 删除约束
MODIFY 字段 类型;
```
### SELECT

```
2. SELECT 字段(聚合函数)
1. FROM 表名 JOIN ... ON 条件
WHERE 条件 AND 不含聚合函数条件
GROUP BY ...
HAYING 聚合函数的条件

3.ORDER BY ...(ASC/DESC)
LIMIT...;
```

1. 别名

```
# 1.空格
SELECT id "ID",...
# 2.AS
SELECT  id, name AS "Name"
```

别名覆盖原名，必须使用别名。先 FROM、WHERE 后 SELECT  

2. 去重

```
SELECT DISTINCT id,...
FROM table;
```

3. 空值（NULL != 0）

空值参与运算结果为空（IFNULL(字段，0)）
- 不好比较
- 效率不高

4. 着重号

自定义字段同关键字冲突使用（``）

5. 常数

字段为常数，结果集固定显示

6. 结构

```
DESCRIBE table; // 显示表中字段信息
DESC table;
```

#### WHERE（过滤）

```
SELECT *
FROM table
WHERE 过滤条件;
```

WHERE 子句紧跟 FROM 子句

#### 算术运算符

SQL 中没有隐式转换和连接作用

| 运算符     | 作用                   |
| ---------- | ---------------------- |
| `+`        | 加                     |
| `-`        | 减                     |
| `*`        | 乘                     |
| `/` or DIV | 除                     |
| `%` or MOD | 取模，同被模数符号相同 |


#### 比较运算符

| 运算符            | 作用               |
| ----------------- | ------------------ |
| `=`               | 是否相等           |
| `<=>`             | 安全等于(NULL)     |
| `<>`or `!=`       | 不等于             |
| `<`               | 小于               |
| `<=`              | 小于等于           |
| `>`               | 大于               |
| `>=`              | 大于等于           |
| IS NULL、ISNULL() | 是否为空           |
| IS NOT NULL       | 是否不为空         |
| LEAST()           | 返回最小值         |
| GREATEST()        | 返回最大值         |
| BETWEEN A AND B   | 是否在 A 与 B 之间 |
| IN (A,B)          | 是否在列表中       |
| NOT IN            | 不属于             |
| LIKE              | 模糊匹配(%,_）     |
| REGEXP、RLIKE     | 正则匹配           |

#### 逻辑运算符

| 运算符 | 作用 |
| ------ | ---- |
| NOT    | 非   |
| OR     | 或   |
| AND    | 与   |
| XOR    | 异或 |

#### ORDER BY (排序)

```
ORDER BY 字段 <ASC/DESC>,...; (默认升序)

```

列别名不能在 WHERE 中使用，可以在 ORDER BY 中使用


#### 分页（LIMIT）

```
LIMIT <偏移量,> 显示记录条数;
LIMIT (pageNo-1)*pageSize,pageSize;
LIMIT 条目数 OFFSET 偏移量;
```

LIMIT 子句置于末尾

#### 多表查询（JOIN...ON）

1. 等值连接 vs 非等值连接
```
NATURAL JOIN # 所有相同字段等值
JOIN table USING(相同字段 d_id) # 指定字段连接 e.d_id = d.d_id
```

2. 自连接 vs 非自连接

3. 内连接 vs 外连接
    

**内连接**：不含不匹配的行
```
JOIN table ON 匹配条件    
```
 
**左外连接**  
```
LEFT OUTER JOIN ... ON ...
```

**右外连接**  
```
RIGHT OUTER JOIN ... ON ...
```

**满外连接**  
```
# FULL OUTER JOIN ... ON ...
```

UNION 返回并集，去重
UNION ALL 返回并集，不去重，效率高

```
SELECL e.id,d.name
FROM e LEFT JOIN d
ON e.id = d.id
UNION ALL
SELECT e.id,d.name
FROM e RIGHT JOIN d
ON e.id = d.id
WHERE e.id IS NULL

SELECL e.id,d.name
FROM e LEFT JOIN d
ON e.id = d.id
WHERE d.id IS NULL
UNION ALL
SELECT e.id,d.name
FROM e RIGHT JOIN d
ON e.id = d.id

SELECL e.id,d.name
FROM e LEFT JOIN d
ON e.id = d.id
WHERE d.id IS NULL
UNION ALL
SELECT e.id,d.name
FROM e RIGHT JOIN d
ON e.id = d.id
WHERE e.id IS NULL

```

**超 3 个表禁用 join**

#### 单行函数

- 只对一行进行变换
- 每行返回一个结果
- 支持嵌套

##### 数值函数

| 函数          | 功能                                  |
| ------------- | ------------------------------------- |
| ABS(X)        | 绝对值                                |
| SIGN(X)       | 返回值正负符号                        |
| PI()          | 圆周率                                |
| CEIL(X)       | `>=x`的最小值                         |
| FLOOR(X)      | `<=x`的最大值                         |
| LEAST(,,,)    | 列表中最小值                          |
| GREATEST(,,,) | 列表中最大值                          |
| MOD(x,y)      | x%y                                   |
| RAND()        | 0~1随机值                             |
| RAND(x)       | 以x为种子                             |
| ROUND(x)      | 最接近x 四舍五入后的数                |
| ROUND(x,y)    | 最接近x 四舍五入后的数，并保留y位小数 |
| TRUNCATE(x,y) | x截断y位小数                          |
| SQRT(x)       | x的平方根                             |

##### 三角函数

| 函数       | 功能       |
| ---------- | ---------- |
| SIN(X)     | 正弦       |
| ASIN(x)    | 反正弦     |
| COS(x)     | 余弦       |
| ACOS(x)    | 反余弦     |
| TAN(x)     | 正切       |
| ATAN(x)    | 反正切     |
| ATAN(m,n)  |            |
| COT(x)     | 余切       |
| RADINS(x)  | 角度->弧度 |
| DEGREES(x) | 弧度->角度 |

##### 指数和对数

| 函数         | 功能          |
| ------------ | ------------- |
| POW(x,y)     | x的y次方      |
| EXP(x)       | e的x次方      |
| LN(x),LOG(x) | e为底x的对数  |
| LOG10(x)     | 10为底x的对数 |
| LOG2(x)      | 2为底x的对数  |

##### 进制转换

| 函数          | 功能           |
| ------------- | -------------- |
| BIN(x)        | 二进制         |
| HEX(x)        | 十六进制       |
| OCT(x)        | 八进制         |
| CONV(x,f1,f2) | f1进制->f2进制 |

##### 字符串函数

| 函数                           | 功能                              |
| ------------------------------ | --------------------------------- |
| ASCII(s)                       | 字符串首字符ASCII码               |
| CHAR_LENGTH(s)                 | 字符串的字符数                    |
| LENGTH(s)                      | 字节数                            |
| CONCAT(s1,s2,...)              | 拼接字符串                        |
| CONCAT_WS(x,s1,s2,..)          | 拼接字符串，串间加x               |
| INSERT(str,idx,len,replacestr) | str从idx开始的len个字符替换为pstr |
| REPLACE(str,a,b)               | b替换a                            |
| UPPER(s)                       | 转换大写                          |
| LOWER(s)                       | 转换小写                          |
| LEFT(s,n)                      | 最左边n个字符                     |
| RIGHT(s,n)                     | 最右边n个字符                     |
| LPAD(str,len,pad)              | 使用pad从左边填充至长度为len      |
| RPAD(str,len,pad)              | 使用pad从右边填充至长度为len      |
| LTRIM(s)                       | 去除左侧空格                      |
| RTRIM(s)                       | 去除右侧空格                      |
| TRIM(s)                        | 去除首尾空格                      |
| TRIM(s1 FROM s)                | 去除开始与结尾的s1                |
| TRIM(LEADING s1 FROM s)        | 去除开始的s1                      |
| TRIM(TRAILING s1 FROM s)       | 去除结尾的s1                      |
| REPEAT(s,n)                    | 返回s重复n次的结果                |
| SPACE(n)                       | 返回n个空格                       |
| STRCMP(s1,s2)                  | 比较s1,s2大小                     |
| SUBSTR(s，idx,len)             | 从idx开始的len个字符              |
| LOCATE(substr,str)             | substr首次出现的位置              |
| ELT(m,s1,s2,..)                | 返回指定位置的字符串              |
| FIELD(s,s1,...)                | 返回s在字符串列表中首次出现位置   |
| FIND_IN_SET(s1,s2)             | s1在s2中出现的位置                |
| REVERSE(s)                     | 反转字串                          |
| NULLIF(val1,val2)              | 比较字符串，相等返回NULL,否则val1 |

##### 日期和时间函数

| 函数                              | 功能                                           |
| --------------------------------- | ---------------------------------------------- |
| CURDATE()                         | 返回当前年、月、日                             |
| CURTIME()                         | 返回当前时、分、秒                             |
| NOW()                             | 当前系统日期，时间                             |
| UTC_DATE()                        | UTC日期                                        |
| UTC_TIME()                        | UTC时间                                        |
| UNIX_TIMESTAMP()                  | 时间戳形式返回当前时间                         |
| UNIX_TIMESTAMP(date)              | 时间->时间戳                                   |
| FROM_UNIXTIME(time)               | 时间戳->时间                                   |
| YEAR(date)/MONTH(date)/DAY(date)  | 具体日期                                       |
| HOUR(tiem)/MINUTE(t)/SECOND(t)    | 具体时间值                                     |
| MONTHNAME(date)                   | 月份                                           |
| DAYNAME(date)                     | 星期几                                         |
| WEEKDAY(date)                     | 周几(0~6                                       |
| QUQRTER(date)                     | 季度                                           |
| WEEK(date)                        | 年的第几周                                     |
| DAYOFYEAR(date)                   | 年的第几天                                     |
| DAYOFMONTH(date)                  | 月份的第几天                                   |
| DAYORWEEK(date)                   | 周几(1~7)                                      |
| EXYTACT(type FROM date)           | 指定日期中的特定部分                           |
| TIME_TO_SEC(time)                 | 转换为秒                                       |
| SEC_TO_TIME(seconds)              | 秒转换为小时、分钟和秒的时间                   |
| DATE_ADD(date,INTERVAL expr type) | 当前时间与给定时间相差INTRTVAL时间段的日期时间 |
| DATE_SUB(date,INTERVAL expr type) | 与date相差INTERVAL时间间隔的日期               |
| ADDTIME(time1,time2)              | 时间相加，秒                                   |
| SUBTIME(t1,t2)                    | 时间相减。秒                                   |
| DATEDIFF(date1,date2)             | date1-date2                                    |
| TIMEDIFF(t1,t2)                   | time1-time2                                    |
| FROM_DAYS(N)                      | 从0000年1月1日，N天以后的日期                  |
| TO_DAYS(date)                     | date距0000年1月1日的天数                       |
| LAST_DAY(date)                    | date所在月份的最后一天                         |
| MAKEDATE(year,n)                  | 给定年份与年份天数的日期                       |
| Maketime(h,m,s)                   | 组合时间                                       |
| PERIOD_ADD(tiem,n)                | time+n                                         |
| DATE_FORMAT(date,fmt)             | 格式化日期                                     |
| TIME_FORMAT(time,fmt)             | 格式化时间                                     |
| GET_FORMAT(daty_type,format_type) | 日期的显示格式                                 |
| STR_TO_DATE(str,fmt)              | 解析字符串为日期                               |

| type               | 含义         |
| ------------------ | ------------ |
| MICROSECOND        | 毫秒         |
| SECOND             | 秒           |
| MINUTE             | 分钟         |
| HOUR               | 小时         |
| DAY                | 天数         |
| WEEK               | 年的第几周   |
| MONTH              | 年的第几月   |
| QUARTER            | 年的第几季度 |
| YEAR               | 年份         |
| SECOND_MICROSECOND | 秒，毫秒     |
| MINUTE_MICROSECOND | 分钟、毫秒   |
| MINUTE_SECOND      | 分钟、秒     |
| ...                | ...          |

| fmt | 说明                     | fmt | 说明                     |
| --- | ------------------------ | --- | ------------------------ |
| %Y  | 4位月份                  | %y  | 2位月份                  |
| %M  | 月份                     | %m  | 2位数字表示月份          |
| %b  | 缩写月份                 | %c  | 数字表示月份             |
| %D  | 英文表示月中天数         | %d  | 2位数字表示月中天数      |
| %e  | 数字表示月中天数         |     |                          |
| %H  | 2位数字表示小时，24      | %h  | 2位数字表示小时，12      |
| %k  | 数字表示小时，24         | %l  | 数字表示小时，12         |
| %i  | 数字表示分钟             | %s  | 数字表示秒               |
| %W  | 星期名称                 | %q  | 星期缩写                 |
| %w  | 数字表示天数             |     |                          |
| %j  | 3位数字表示年中天数      | %U  | 数字表示年中周数，Sunday |
| %u  | 数字表示年中周数，Monday |     |                          |
| %T  | 24小时制                 | %r  | 12小时制                 |
| %p  | AM or PM                 | %%  | %                        |

![sql_fmt_type](https://cdn.jsdelivr.net/gh/Noxryn/PicGallery@main/img/sql_fmt_type.png)

##### 流程控制函数

| 函数                                         | 用法                     |
| -------------------------------------------- | ------------------------ |
| IF(val,v1,v2)                                | val为TRUE,返回v1,否则v2  |
| IFNULL(v1,v2)                                | v1不为NULL,返回v1,否则v2 |
| CASE WHEN 条件1 THEN 结果1 ELSE result END   |                          |
| CASE expr WHEN 常量值1 THEN 值1 ELSE 值n END |                          |

##### 加密解密函数

| 函数             | 用法             |
| ---------------- | ---------------- |
| PASSWORD(str)    | 字符串加密，41位 |
| MD5(str)         | md5加密          |
| SHA(str)         | sha加密          |
| ENCODE(val,pass) | 使用pass加密val  |
| DECODE(val,pass) | 使用pass解密val  |

##### 信息函数

| 函数              | 用法             |
| ----------------- | ---------------- |
| VERSION()         | 版本号           |
| CONNECTION_ID()   | 服务器连接数     |
| DATABASE,SCHEMA() | 命令行当前数据库 |
| USER()            | 连接用户名       |
| CHARSET(val)      | val的字符集      |
| COLLATION(val)    | 比较规则         |

##### 其它函数

| 函数                    | 用法                        |
| ----------------------- | --------------------------- |
| FORMAT(val,n)           | 格式化数字，保留小数点后n位 |
| CONV(val,from,to)       | 进制转换                    |
| INET_ATON(ip)           | ip->数字                    |
| INET_NTOA(val)          | 数字-ip                     |
| BENCHMARK(n,expr)       | 表达式执行n次               |
| CONVERT(val USING code) | 修改字符编码                |

#### 聚合函数

- 输入数据集合
- 输出一个结果

##### 常用函数

| 函数    | 用法               |
| ------- | ------------------ |
| AVG()   | 平均               |
| SUM()   | 求和               |
| MAX()   | 最大值             |
| MIN()   | 最小值             |
| COUNT() | 查询计数，不计NULL |

##### 分组（GROUP BY）

```
GROUP BY 字段 WITH ROLLUP # 统计记录数量，与ORDER BY 互斥
```

***SELECT 中出现的非组函数字段必须声明在GROUP BY中**

##### 过滤（HAVING）

**WHERE 不支持聚合函数**

HAVING 依托于 GROUP BY

WHERE 效率高于 HAVING  

```
HAVING 条件
```

### INSERT

```
INSERT INTO <table>
VALUES (val1,val2,...);

INSERT INTO <table(val1,val2,..)>
VALUES (val1,val3,...),
VALUES (val2,val4,...)

INSERT INTO <table(val1,val2,..)>
<查询语句 SELECT>
```

### UPDATE

```
UPDATE <table>
SET <字段> = ... 
WHERE 条件;
```

### DELETE

```
DELETE FROM <table>
WHERE 条件;
```

### 计算列

```
CREATE TABLE test1(
    a INT,
    b INT,
    c INT GENERATED ALWAYS AS (a+b) VIRTUAL # 计算列
)；
```

## 视图

**存储起来的 SELECT 语句**

- 简化查询，提升效率
- 控制数据访问
- 视图本身的增删不影响数据，对对数据的更改影响基表数据
- 减少数据冗余

```
# 创建视图
CREATE VIEW 视图名称（字段别名,...）
AS 查询语句

# 数据格式化
SELECT CONCAT(name,'(',id,')') AS dept

# 查看视图
SHOW TABLES;

# 查看视图结构
DESCEIBE <veiw_name>;

# 查看视图属性
SHOW TABLE STATUS LINK 'veiw_name';

# 查看视图的详细信息
SHOW CREATE VIEW <view_name>;

# 修改视图
CREATE OR REPLACE VIEW <name>(字段，...)
AS 查询子句

ALTER VIEW <name>
AS 查询子句
 
# 删除视图
DROP VIEW <name>;
```

## 存储过程

一组预先编译的 SQL 语句的封装  
- 简化操作
- 提高效率
- 减少网络传输
- 提高数据查询的安全性
  
### 分类

- 没有参数
- IN
- OUT
- IN AND OUT
- INOUT

### 语法 

```
DELIMITER $ # 自定义分隔符
CREATE PROCEDURE 存储过程名(IN|OUT|INOUT 参数名 参数类型,...)
[约束条件]
/*
LANGUAGE SQL # 执行体为 SQL
    [NOT] DETERMINISTIC # 执行结果是否确定，相同输入是否输出相同
    # 不含读写数据的 SQL | 不含 SQL | 包含读数据的 SQL | 包含写数据的 SQL
    {CONTAINS SQL|NO SQL|READS SQL DATA|MODIFIES SQL DATA} 
                # 仅创建者|具有访问权限用户
    SQL SECURITY {DEFINER|INVOKER}
    # 注释信息
    COMMENT 'string'
*/
BEGIN
         INTO 参数名 # OUT
END $

DELIMITER ;

# 查看输出
SELECT @参数名;

# 调用
CALL 存储过程名(参数);

SET @变量= 参数;
CALL 存储过程名(@变量);

# 查看存储过程/函数
SHOW CREATE {PROCEDURE|FUNCTION} <name>;
SHOW  {PROCEDURE|FUNCTION} STATUS LIKE '';
information_schema.Routines表

# 修改
ALTER {PROCEDURE|FUNCTION} <name>
[约束条件]
COMMENT '注释';

# 删除
DROP {PROCEDURE|FUNCTION}  [IF EXISTS] <name>;
```

## 存储函数

```
SET GLOBAL log_bin_trust_function_creators = 1; # 保障函数创建成功
DELIMITER $ 
CREATE FUNCTION 函数名（参数名 参数类型,...)
RETURNS 返回类型
[约束条件]
/*
LANGUAGE SQL # 执行体为 SQL
    [NOT] DETERMINISTIC # 执行结果是否确定，相同输入是否输出相同
    # 不含读写数据的 SQL | 不含 SQL | 包含读数据的 SQL | 包含写数据的 SQL
    {CONTAINS SQL|NO SQL|READS SQL DATA|MODIFIES SQL DATA} 
                # 仅创建者|具有访问权限用户
    SQL SECURITY {DEFINER|INVOKER}
    # 注释信息
    COMMENT 'string'
*/
BEGIN
    RETURN  (查询语句);
END
DELIMITER ;

# 调用
SELECT 函数名();
```

## 变量

```
SHOW GLOBAL VARIABLES LINK ''; # 查看全局变量 重启失效  
SHOW SESSION VARIABLES LINK ''; # 查看会话变量

SELECT @@{global/session}.变量名; # 查看指定变量

SET {GLOBAL/SESSION} 变量名 = 值;
SET PERSIST GLOBAL 变量名 = 值; # 变量持久化

SET @变量名=值; # 会话用户变量
SELECT @变量名; # 查看变量
SWLECT 表达式 INTO 变量名; # 赋值 

# 局部变量
BEGIN 
    DECLARE 变量名 数据类型 [DEFAULT 默认值];
    select 变量名;
END
```

## 错误处理

```
DECLARE 错误名称 CONDITION FOR {错误码| SQLSTATE 错误条件}

#               继续执行|退出|撤回
DECLARE 处理方式{CONTINUE|EXIT|UNDO} HANDLER FOR {错误名称|SQLWARNING|NOT FOUND|SQLEXCEPTION} 处理语句
```

## 流程控制 

### 分支

```
IF 表达式 THEN 操作
ELSEIF 表达式 THEN 操作
ELSE 操作
END IF;

CASE [表达式]
WHEN 值 THEN 结果
WHEN 值 THEN 结果
ELSE 结果
END CASE;
```

### 循环

```
标签:LOOP  # 无条件循环
    LEAVE 标签; # break
    ITERATE 标签; # continue
END LOOP 标签;

标签：WHILE 条件 DO # 先判断后执行
    /**/
END WHILE 标签;

标签：REPEAT # 先执行后判断
    /**/
UNTIL 结束条件
END REPEAT 标签;
```

## 游标（指针）

```
DECLARE 游标名 CURSOR FOR 查询语句; # 声明游标
OPEN 游标名; # 打开游标
FETCH 游标名 INTO 变量; # 使用游标 范围for
CLOSE 游标名; # 关闭游标
```

## 触发器

- 保证数据的完整性 
- 事件触放
- 记录操作日志
- 进行数据合法性检查
- 可读性差

```
CREATE TRIGGER 触发器名称
# 触放时间 触放事件
{BRRORE|AFTER} {INSERT|UPDATE|DELETE} ON 表名
FOR EACH ROW
BEGIN  # 执行语句块

END

SHOW TEIGGRES; # 查看触发器爹娘关于
SHOW CREATE TEIGGER 触发器名;
information_schema.TRIGGERS;

DROP TRIGGER IF EXISTS 触发器名;
``` 
  
## 窗口函数

## 公共表达式

# 高级篇

## Linux 环境配置

### 卸载

```
mysql --version # 查看版本
systemctl status mysql # 查看启动状态
systemctl stop mysql # 关闭 mysql 服务
sudo apt-get purge mysql-server mysql-server-8.0 mysql-common # 卸载软件包及依赖
sudo rm -rf /etc/mysql /var/lib/mysql # 删除配置文件
sudo apt-get autoremove # 清理残留文件
sudo apt-get autoclean
mysql --version # 验证卸载结果
```

### 安装

```
# 安装
sudo apt install mysql-server
# 启动
sudo systemctl start mysql
# 开机自启
sudo systemctl enable mysql
# 状态
sudo systemctl status mysql
# 开发头文件和动态库文件
sudo apt install -y libmysqlclient-dev
```

### 字符集

- 服务器级别
- 数据库级别
- 表级别
- 列级别

## 用户管理

```
CREATE USER 用户名 [IDENTIFIED BY '密码'][,用户名 [IDENTIFIED BY '密码']]; # 创建用户

UPDATE mysql.user SET USER= <> WHERE USER= <>; # 修改用户名
FLUSH PRIVILEGES;   # 修改生效

DROP USER user[,user]… # 删除用户

ALTER USER user IDENTIFIED BY 'new_password'; # 修改密码
SET PASSWORD FOR 'username'@'hostname'='new_password';

```

## 权限管理

1. CREATE 和 DROP：可以创建新的数据库和表，或删除已有的数据库和表
2. SELECT、INSERT、UPDATE 和 DELETE：允许在一个数据库现有的表上实施操作
3. SELECT：从一个表中检索行
4. INDEX：允许创建或删除索引
5. ALTER：使用ALTER TABLE来更改表的结构和重新命名表
6. CREATE ROUTINE：创建保存的程序（函数和程序）
7. ALTER ROUTINE：更改和删除保存的程序
8. EXECUTE：执行保存的程序
9. GRANT：允许授权给其他用户，可用于数据库、表和保存的程序
10. FILE：能读或写MySQL服务器上的任何文件

- 最小权限
- 限制登录
- 安全密码
- 定时清理

```
GRANT 权限1,权限2,…权限n ON 数据库名称.表名称 TO 用户名@用户地址 [IDENTIFIED BY ‘密码口令’];

SHOW GRANTS;
# 或
SHOW GRANTS FOR CURRENT_USER; 
# 或 
SHOW GRANTS FOR CURRENT_USER()

REVOKE 权限1,权限2,…权限n ON 数据库名称.表名称 FROM 用户名@用户地址;

CREATE ROLE 'role_name'[@'host_name'] [,'role_name'[@'host_name']]...
GRANT privileges ON table_name TO 'role_name'[@'host_name'];
SHOW PRIVILEGES\G;

REVOKE privileges ON tablename FROM 'rolename';
DROP ROLE role [,role2]...
GRANT role [,role2,...] TO user [,user2,...];
SET DEFAULT ROLE ALL TO 'kangshifu'@'localhost';
```


## InnoDB （事务存储引擎）—— 表结构

- 具备外键支持
- 支持事务，可以回滚，崩溃恢复
- 更新、删除效率高
- 处理巨大数据量
- 内存要求高，处理效率差
- 行锁
- 高并发
  
## MyISAM (非事务存储引擎) 
  
  - 不支持事务，行级锁，外键
  - 崩溃后无法安全恢复
  - 不支持行锁，不利于并发
  - 访问速度快
  - 高查询

## 索引优化

- 字段不能过长
- 主键单调

## 索引

### 分类

- 普通索引
- 唯一性索引：UNIQUE，索引值唯一含空
- 主键索引：特殊唯一性索引，不为空，只有一个主键索引, PRIMARY KEY
- 全文索引：分词，FULLTEXT，字符串(CHAR、VARCHAR、TEXT)
- 单列索引：单个字段进行索引
- 多列索引、联合索引：多个字段进行索引
- 空间索引：SPATIAL，空间类型
  
- 聚族索引：主键
- 二级索引：非主键
  
```
CREATE TABLE table { # 创建表时隐式生成索引，约束->索引
    id INT PRIMARY KEY,
    name VARCHAR(20)
}

CREATE TABLE table_name [col_name,data_type)
# 唯一索引|全文索引|空间索引     同义，索引  索引名      索引字段   字符串类型指定长度  
[UNIQUE|FULLTEXT|SPATIAL] INDEX|KEY  [indsx_name] (col_name [length]) [ASC|DESC]


SHOE CREATE TABLE name;
SHOW INDEX FROM name;

ALTER TABLE table
ADD INDEX 索引名（字段）

CREATE [] INDEX 索引名 ON table(字段)

ALTER TABLE table
DROP INDEX idx

DROP INDEX id ON table

# 隐藏索引，软删除 invisible/visible

SET SESSION optimizer_swith = "use_invisible_indexes= on"; # 隐藏索引对优化器可见
```

### 适合

- 字段具有唯一性限制
- 频繁作为过滤条件判断的字段
- 经常 GROUP BY 和 ORDER BY 
- UPDATE、DELETE的 WHRER条件列
- DISTINCT字段
- 多表连接的连接字段
- 类型小的列
- 字符串前景
- 区分度高的列
- 使用频繁的列
- 联合索引优于单值索引

### 限制

- where中不使用的字段
- 数据量小的表
- 大量重复数据的列
- 表经常更新
- 无序值
- 删除使用少的值
- 冗余或重复索引

## 追踪

观察服务器状态->周期性波动 ——》更改缓存-》
是否仍有卡顿-》开启慢查询-》EXPLANIN SHOW PROFILING-》
执行时间长-> 索引优化，JOIN过多，数据表设计-》
等待时间长-》调优服务器参数-》查询是否凭颈-》读写分离，分库分表

###  性能参数

```
SHOW [GLOABL|SESSION] STATUS LIKE ;
```
- Connections:连接次数
- Uptime:上线时间
- Slow_queries:慢查询次数
- Innodb_rows_read:查询返回行数
- Innodb_rows_inserted:插入行数
- Innodb_rows_updated:更新行数
- Innodb_rows_inserted:删除行数
- Com_select:查询次数
- Com_insert:插入次数
- Com_uupdate:更新次数
- Com_delete:删除次数
  
### 查询成本 Last_query_cost

```
SHOW STATUS LIKE 'last_query_cost'
```

### 慢查询 （10s）

```
# 开启慢查询日志 
show varibles like '%slow_query_log'
set slow_query_log = on; 
# 阈值
show variables like 'long_query_time' # 还有最小扫描记录 0
set gloable long_query_tiem = （s）
set long_query_tiem = （s）


# 生成日志
mysqladmin -u root ip flush-logs slow

```

### 成本查看 SHOW PROFILE——information_schema中的profiling表

```
set profiling = on
show profiles [for <id>]
```
- ALL
- BLOCK IO：块 IO
- CONTEXT SWITCHES：上下文切换
- CPU
- IPC：发送和接收
- MEMORY
- PAGE FAULTS：页面错误
- SOURCE：source_%相关
- SWAPS：交换次数

优化注意
1. converting HEAP to MyISAM：内存不足
2. Creating tmp table:创建临时表
3. Copying to tmp table on disk：内存中临时表复制到磁盘
4. locked

### EXPLAIN

```
EXPLAIN 语句；
DESCRIBE 语句
```
trace

## 优化

物理优化- 索引 ，表连接
逻辑优化- sql

### 索引失效

是否使用索引——版本、数据量、数据选择度


左外连接

小表驱动，被驱动表加索引，块嵌套、hash JON. strainght_join(固定驱动表)



不建议使用子查询，建议将子查询SQL拆开结合程序多次查询，或使用 JOIN 来代替子查询。

index排序，fileshot排序-在order by排序

覆盖索引