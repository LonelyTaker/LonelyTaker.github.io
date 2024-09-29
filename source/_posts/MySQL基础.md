---
title: MySQL基础
date: 2024-09-29 18:35:36
categories: 后端
tags: MySQL
---

# DDL

## 数据库

* 创建数据库

  ```sql
  CREATE DATABASE IF NOT EXISTS dbtest CHARACTER SET 'utf8'
  # 显示创建数据库时的语句
  SHOW CREATE DATABASE dbtest
  ```

* 删除数据库

  ```sql
  DROP DATABASE dbtest
  ```

## 表

* 创建表

  ```sql
  CREATE TABLE 表名 IF NOT EXISTS (
   字段信息
  )
  # 显示创建表时的语句
  SHOW CREATE TABLE 表名
  # 展示表结构
  DESC 表名
  
  # 计算列(MySQL8特性)
  CREATE TABLE test (
   a INT,
   b INT,
   c INT GENERATED ALWAYS AS (a+b) VIRTUAL
  )
  ```

* 删除表

  ```sql
  DROP TABLE 表名
  ```

* 修改表

  ```sql
  ALTER TABLE 表名
      ADD 字段	# 添加字段
      MODIFY 字段 # 修改字段
  	DROP COLUMN 字段 # 删除字段
  ```

* 清除表数据

  ```
  TRUNCATE 表名
  ```



>DROP、DELETE、TRUNCATE三者的区别？
>
>1. DROP和TRUNCATE属于DDL定于语言，执行后无法回滚，DELETE属于DML操作语言，会走事务，执行时会触发触发器，可被回滚
>2. DELETE不会立刻释放磁盘空间，DROP和TRUNCATE会
>3. DROP用于删除库和表，包括表结构，DELETE、TRUNCATE用于清除表数据
>4. 从执行速度上来说，DROP > TRUNCATE > DELETE
>
>https://zhuanlan.zhihu.com/p/270331768



# DML

## 增

```sql
INSERT INTO 表名(字段信息)
VALUES
(数据1),
(数据2);

# 从查询中获取数据插入
INSERT INTO 表名(字段1,字段2)
SELECT 字段1,字段2 # 查询的字段一定要和添加到表的字段一一对应
FROM 表
```

## 删

```sql
DELETE FROM 表名 WHERE ...

# 联表删除
DELETE m,u FROM employees m 
JOIN users u 
ON m.userid = u.userid 
WHERE m.userid = 'Bbiri'
```

## 改

```sql
UPDATE 表名 SET ...,... WHERE ...

UPDATE empl SET salary = salary * 1.2, hire_date = CURDATE()
WHERE `name` LIKE '%a%'
```



> DML操作默认情况下，执行完都会自动提交数据
> 如果希望执行完后不自动提交，则需要使用 SET autocommit = FALSE



# 数据类型

## 数值类型

### 整数类型

* TINYINT 1字节 （一般用于枚举数据，比如系统设定取值范围很小且固定场景）
* SMALLINT 2字节 （用于较小范围的统计数据，如工厂的固定资产库存等）
* MEDIUMINT 3字节 （用于较大整数计算，如车站客流量）
* INT 4字节 （取值范围足够大，一般不用考虑超限问题，如商品编号）（使用最多）
* BIGINT 8字节 （处理巨大整数才会用到。如大型门户网站的点击量等）

**可选属性：**

* M: 宽度（MySQL8.0后不推荐）

  ```sql
  CREATE TABLE test(
   f1 INT,
   f2 INT(5),
   f3 INT(5) ZEROFILL 
   # 显示宽度5，当insert的值不足5位时，拿0填充；
   # 当使用 ZEROFILL 时，会自动添加 UNSIGNED
  )
  ```

* UNSIGNED: 无符号（正数）

### 浮点类型

* FLOAT 4字节
* DOUBLE 8字节

MySQL允许使用非标准语法 ：FLOAT(M,D) M-精度 D-标度 M=整数位+小数位 D=小数位 D<=M<=255 0<=D<=30

浮点类型也可以加 UNSIGNED ，但是不会改变数据范围，无符号取值范围相当于有符号取值范围的一半。这是因为MySQL存储浮点格式为：符号、尾数、阶码。无论有没有符号，浮点数都会存储表示符号的部分。因此无符号取值范围相当于有符号取值范围大于0的部分

如果不指定精度，则按照实际精度（由操作系统决定）显示

如果小数位长度超过，则会四舍五入；如果整数位长度超过，则会报错

### 定点数类型

* DECIMAL(M,D) M+2字节

在存储同样范围的值时，DECIMAL通常使用更多的空间，float使用4个字节存储，double使用8个字节，而DECIMAL依赖于M和D的值

### 位类型

* BIT(M) 1<=M<=64 占用约为(M+7)/8字节

使用SELECT查询位字段时，可以用`BIN()`或者`HEX()`函数查看

```sql
BIN() # 2进制
HEX() # 16进制
```



## 日期与时间类型

### YEAR类型

以4位字符串或数字格式表示YEAR类型，格式YYYY，最小值1901，最大值2155

以2位字符串或数字表示（不推荐）

### DATE类型

日期，不包括时间，需要3个字节存储空间

格式：YYYY-MM-DD

```sql
CURRENT_DATE()
CURDATE()
```

以下都可以写入

```sql
'2020-10-01' '20201001' 20201001
```

### TIME类型

时间，需要3个字节存储空间

格式：HH:MM:SS

* 可以插入天数 如`'D HH:MM:SS'`，D会转化为小时，计算格式为`D*24+HH`

* 如果带冒号的【字符串】表示时间，如`'12:10'`，表示`'12:10:00'`

* 如果不带冒号的【字符串或数字】表示时间，如`1210`，MySQL会将右边两位解析成秒，表示`'00:12:10'`

* 如果插入一个不合法的字符串或数字，会自动转为`'00:00:00'`存储

```sql
CURRENT_TIME()
CURTIME()
```

### DATETIME类型 

日期+时间的组合，需要8个字节存储空间

格式：YYYY-MM-DD HH:MM:SS 

* 以`YYYY-MM-DD HH:MM:SS`格式或者`YYYYMMDDHHMMSS`格式【字符串】插入时，最小值`'1000-01-01 00:00:00'`，最大值`'9999-12-03 23:59:59'`

* 以`YYYYMMDDHHMMSS`格式【数字】插入时，会被转为`YYYY-MM-DD HH:MM:SS`格式

### TIMESTAMP类型

时间戳，格式与DATETIME类型相同，只需要4字节存储，但时间范围小，只能存储 '1970-01-01 00:00:01 UTC' 到 '2038-01-19 03:14:07 UTC' 之间

使用TIMESTAMP存储的同一个时间，在不同时区查询会显示不同时间

>TIMESTAMP与DATETIME区别：
>
>* TIMESTAMP存储空间小，表示范围也比较小
>* 底层存储方式不同，TIMESTAMP存储的是毫秒值
>* 在计算日期或者比较大小时，TIMESTAMP更方便、更快
>* TIMESTAMP与时区相关。TIMESTAMP会根据用户的时区不同显示不同的结果。而DATETIME只能反映插入时当地时区，其他时区的人查看数据会有误差

>开发经验：
>
>DATETIME使用最多，但是存储注册时间、商品发布时间等，推荐使用TIMESTAMP，更便于计算



## 文本字符串类型

### CHAR/VARCHAR类型

| 类型       | 特点     | 长度 | 长度范围    | 占用空间         |
| ---------- | -------- | ---- | ----------- | ---------------- |
| CHAR(M)    | 固定长度 | M    | 0<=M<=255   | M个字节          |
| VARCHAR(M) | 可变长度 | M    | 0<=M<=65535 | 实际长度+1个字节 |

* VARCHAR最大有效长度由最大行大小和使用的字符集确定（65535并不准确）
* 如果保存时数据实际长度比CHAR类型声明长度小，则会在`右侧填充空格`以达到指定长度。当MySQL检索CHAR类型数据时，会去除尾部空格
* 检索VARCHAR类型数据时，会保留尾部空格

| 类型       | 空间         | 时间   | 适用场景             |
| ---------- | ------------ | ------ | -------------------- |
| CHAR(M)    | 浪费存储空间 | 效率高 | 存储不大，速度要求高 |
| VARCHAR(M) | 节省存储空间 | 效率低 | 非CHAR的情况         |

情况1：存储较短的信息，使用CHAR

情况2：存储固定长度信息，使用CHAR

情况3：频繁修改的字段，使用CHAR

情况4：不同存储引擎中的情况：

* MyISAM存储引擎建议使用CHAR类型
* MEMORY存储引擎建议使用CHAR类型
* InnoDB存储引擎建议使用VARCHAR类型

### TEXT类型

| 类型       | 长度 | 长度范围         | 占空空间  | 特点               |
| ---------- | ---- | ---------------- | --------- | ------------------ |
| TINYTEXT   | L    | 0<=L<=255        | L+2个字节 | 可变长度，小文本   |
| TEXT       | L    | 0<=L<=65535      | L+2个字节 | 可变长度，文本     |
| MEDIUMTEXT | L    | 0<=L<=16777215   | L+3个字节 | 可变长度，中等文本 |
| LONGTEXT   | L    | 0<=L<=4294967295 | L+4个字节 | 可变长度，长文本   |

* 由于实际存储长度不确定，MySQL不允许TEXT类型作为主键

* 实际开发中，如果存入内容不是特别大，建议使用CHAR，VARCHAR
* TEXT类型和BLOB类型删除后容易导致“空洞”，使得文件碎片比较多，所以频繁使用的表不建议包含TEXT类型，建议单独分出去

### ENUM类型

| 类型 | 长度 | 长度范围    | 占空空间   |
| ---- | ---- | ----------- | ---------- |
| ENUM | L    | 0<=L<=65535 | 1或2个字节 |

* 当ENUM类型包含1~255个成员时，需要1个字节存储
* 当ENUM类型包含256~65535个成员时，需要2个字节存储

```sql
CREATE TABLE test(
 season ENUM('春','夏','秋','冬'，'unknow')
)

INSERT INTO test(season)
VALUES
('春'),
('UNKNOW'), # 忽略大小写
(1), # 可以使用索引
('1');
```

### SET类型

| 类型 | 长度 | 长度范围 | 占空空间                                                     |
| ---- | ---- | -------- | ------------------------------------------------------------ |
| SET  | L    | 0<=L<=64 | 1<=L<=8 1个字节<br />9<=L<=16 2个字节<br />17<=L<=24 3个字节<br />25<=L<=32 4个字节<br />33<=L<=64 8个字节 |

```sql
CREATE TABLE test(
 s ENUM('A','B','C')
)

INSERT INTO test(s)
VALUES
('A'),
('A,B'),
('A,B,C,A'), # 插入重复的SET类型成员时，会自动去重
('A,B,C,D'); # 插入不在SET内元素时会报错
```



## 二进制类型

### BINARY/VARBINARY类型

类型CHAR和VARCHAR，只是他们存储的是二进制字符串

| 类型         | 特点     | 长度范围    | 占用空间         |
| ------------ | -------- | ----------- | ---------------- |
| BINARY(M)    | 固定长度 | 0<=M<=255   | M个字节          |
| VARBINARY(M) | 可变长度 | 0<=M<=65535 | 实际长度+1个字节 |

### BLOB类型

可以存储一个二进制的大对象，比如图片、音频、视频等

| 类型       | 长度 | 长度范围                      | 占空空间  |
| ---------- | ---- | ----------------------------- | --------- |
| TINYBLOB   | L    | 0<=L<=255                     | L+1个字节 |
| BLOB       | L    | 0<=L<=65535（相当于64KB）     | L+2个字节 |
| MEDIUMBLOB | L    | 0<=L<=16777215（相当于16MB）  | L+3个字节 |
| LONGBLOB   | L    | 0<=L<=4294967295（相当于4GB） | L+4个字节 |

需要注意的是，实际开发中往往不会在数据库中存储大对象数据

在使用TEXT和BLOB时需要注意以下几点

1. BLOB和TEXT在执行了大量的删除和更新操作时，会在数据表中留下很多“空洞”。为了提高性能，建议定期使用OPTIMIZE TABLE功能对表进行`碎片整理`
2. 如果需要对大文本字段进行模糊查询，MySQL提供了`前缀索引`
3. 建议把BLOB或TEXT列分离到单独的表中



## JSON类型

在MySQL 8.x版本中，JSON类型提供了可以进行自动验证的JSON文档和优化的存储结构，使得在MySQL中存储和读取JSON类型的数据更加方便高效

```sql
CREATE TABLE test(
 js JSON
);

INSERT INTO test(js)
VALUES ('{"name":"xiaoming"}')
```

当需要检索JSON类型的字段中某个具体值时，可以使用`->`和`->>`符号

```sql
SELECT js -> '$.name' AS j_name, js -> '$.age' AS j_age, js -> '$.address.province' AS j_province
FROM test
```



# 约束

```sql
# 查看表中约束
SELECT * FROM information_schema.table_constraints
WHERE table_name = '表名';
```

约束类型：

* 列级约束

  主键约束、外键约束、唯一约束、检查约束、默认约束、非空约束、自增列约束

* 表级约束

  主键约束、外键约束、唯一约束、检查约束



## 非空约束

关键字：NOT NULL

```sql
# 在建表的时候添加约束
CREATE TABLE test1(
 id INT NOT NULL,
 last_name VARCHAR(15) NOT NULL,
 email VARCHAR(25),
 salary DECIMAL(10,2)
);

# 在修改时添加约束
ALTER TABLE test1
MODIFY email VARCHAR(25) NOT NULL;

# 在修改时删除约束
ALTER TABLE test1
MODIFY email VARCHAR(25) NULL;
```



## 唯一性约束

关键字：UNIQUE

特点：

* 同一个表可以有多个唯一约束
* 唯一约束可以是某一个列值唯一，也可以多个列组合值唯一
* 唯一性约束允许列值为空，并且允许多个为空值
* 在创建唯一约束时如果不命名，默认和列名相同
* MySQL会给唯一约束的列默认创建一个唯一索引

```sql
# 在建表的时候添加约束
CREATE TABLE test2(
 id INT UNIQUE, # 列级约束
 last_name VARCHAR(15),
 email VARCHAR(25),
 salary DECIMAL(10,2),
    
 # 表级约束
 CONSTRAINT uk_test2_email UNIQUE(email)
);

# 在修改时添加约束
# 方式1
ALTER TABLE test2
MODIFY email VARCHAR(25) UNIQUE;
# 方式2
ALTER TABLE test2
ADD CONSTRAINT uk_test2_sal UNIQUE(salary);

# 复合的唯一性约束
CREATE TABLE user(
 id INT,
 `name` VARCHAR(15),
 `password` VARCHAR(25),

 CONSTRAINT uk_user_name_pwd UNIQUE(`name`,`password`)
 # UNIQUE(`name`,`password`)
);

# 删除唯一性约束
-- 添加唯一性约束的列上也会自动创建唯一索引
-- 删除唯一约束只能通过删除唯一索引的方式删除
-- 删除时需要指定唯一索引名，唯一索引名和唯一约束名一致
-- 如果创建唯一约束时未指定名字，如果是单列，默认和列名相同；如果是组合列，默认和()中排在第一个的列名相同
ALTER TABLE user
DROP INDEX uk_user_name_pwd;
```



## 主键约束

关键字：PRIMARY KEY

特点： 

* 相当于唯一约束+非空约束的组合
* 一个表最多一个主键约束
* 主键约束对应表中的一列或多列（复合主键）
* 如果是多列组合的复合主键约束，这些列都不允许为空值，且组合的值不允许重复
* MySQL的主键约束名总是PRIMARY，无法自定义
* 创建主键约束时，系统会自动创建对应的主键索引。如果删除主键约束，对应索引也会自动删除
* 不要修改主键字段值。如果修改了主键的值，有可能会破坏数据完整性

```sql
# 在建表的时候添加约束
CREATE TABLE test3(
 id INT PRIMARY KEY, # 列级约束
 last_name VARCHAR(15),
 email VARCHAR(25),
 salary DECIMAL(10,2),
 
 # 表级约束
 # CONSTRAINT uk_test3_id PRIMARY KEY(id) # 自定义约束名无效，仍然为PRIMARY
 # PRIMARY KEY(last_name,email)
);

# 在修改时添加约束
ALTER TABLE test3
ADD PRIMARY KEY(id);

# 删除主键约束
ALTER TABLE test3
DROP PRIMARY KEY;
```



## 自增列

关键字：AUTO_INCREMENT

特点：

* 一个表最多一个自增长列
* 当需要产生唯一标识符或顺序值时，可设置自增长
* 自增长列约束的列必须是键列（主键列，唯一键列）
* 自增约束的列数据类型必须是整数类型
* 如果自增列指定了0和null，会在当前最大值的基础上自增；如果自增列手动指定了具体值，直接赋值为具体值

```sql
# 在建表的时候添加约束
CREATE TABLE test4(
 id INT PRIMARY KEY AUTO_INCREMENT,
 last_name VARCHAR(15),
 email VARCHAR(25),
 salary DECIMAL(10,2),
);

# 在修改时添加约束
ALTER TABLE test3
MODIFY id INT AUTO_INCREMENT;

# 在修改时删除约束
ALTER TABLE test3
MODIFY id INT;
```

MySQL 8.0将自增主键的计数器持久化到`重做日志`中。数据库重启自增变量不会变；

此前版本放在内存中，数据库重启后自增变量会变



## 外键约束

关键字：FOREIGN KEY

特点：

* 从表的外键列，必须引用主表的主键或唯一约束列
* 在创建外键约束时，如果不给外键约束命名，默认名不是列名，而是自动产生一个外键名
* 创建表时就指定外键约束的话，先创建主表，再创建从表
* 删表时，先删从表（或先删外键约束），再删主表
* 当主表的记录被从表参照时，主编记录不允许被删除，需要先删除从表中依赖该记录的数据，才能删除主表中的数据
* 一个表可以建立多个外键约束
* 从表外键列 和 主表被参照列 名字可以不同，但数据类型必须一样，逻辑意义一致
* 当创建外键约束时，系统默认会在所在的列上建立对应的普通索引。但是索引名是列名，不是外键约束名
* 删除外键约束后，必须手动删除对应的索引

```sql
# 在建表的时候添加约束
# 建立主表
CREATE TABLE dept1(
    dept_id INT PRIMARY KEY AUTO_INCREMENT,
    dept_name VARCHAR(15)
);
# 建立从表
CREATE TABLE emp1(
    emp_id INT PRIMARY KEY AUTO_INCREMENT,
    emp_name VARCHAR(15),
    dept_id INT
    
    CONSTRAINT fk_emp1_dept_id FOREIGN KEY(dept_id) REFERENCES dept1(dept_id)
);

# 添加约束
ALTER TABLE emp1
ADD CONSTRAINT fk_emp1_dept_id FOREIGN KEY(dept_id) REFERENCES dept1(dept_id)

# 删除约束
# 1.先删除外键约束
ALTER TABLE 从表名
DROP FOREIGN KEY 外键约束名
# 2.再删除索引
ALTER TABLE 从表名
DROP INDEX 索引名
```



## 检查约束

关键字：CHECK

特点：

* MySQL 8.0中可以使用。5.7不支持，虽然可以添加，但无任何作用

```sql
CREATE TABLE test(
 id INT PRIMARY KEY AUTO_INCREMENT,
 age INT CHECK(age > 20),
 sex CHAR(2) CHECK(sex in ('男','女'))
);

# 删除约束
ALTER TABLE test
DROP CHECK 检查约束名;
```



## 默认值约束

关键字：DEFAULT

```sql
# 在建表的时候添加约束
CREATE TABLE test(
 id INT PRIMARY KEY AUTO_INCREMENT,
 age INT DEFAULT 20
);

# 在修改时添加约束
ALTER TABLE test
MODIFY age INT DEFAULT 20;

# 删除约束
ALTER TABLE test
MODIFY age INT;
```



>非空约束、默认值约束、自增列约束可以在ALTER TABLE中用MODIFY修改（添加/删除）
>
>其余约束需要注意修改方式，MODIFY/ADD/DROP



# 视图

* 视图是一种虚拟表，本身不具有数据
* 视图建立在已有的表上，视图赖以建立的表称为基表
* 视图的创建和删除只影响视图本身，不影响基表。但是对视图中的数据进行增删改时，基表数据也会相应变化，反之亦然
* 向视图提供数据内容的语句为SELECT语句，可以将视图理解为存储起来的SELECT语句



## 创建视图

```sql
# 创建视图
CREATE VIEW 视图名
AS
SELECT ...

# 确定视图中字段名的方式1
CREATE VIEW 视图名
AS
SELECT 字段1 AS 别名1, 字段2 AS 别名2, 字段3 AS 别名3
FROM ...
# 确定视图中字段名的方式2
CREATE VIEW 视图名(别名1, 别名2, 别名3)	# 字段个数需对应
AS
SELECT 字段1, 字段2, 字段3
FROM ...
```



## 查看视图

```sql
# 查看数据库的表对象、视图对象
SHOW TABLES;
# 查看视图结构
DESC 视图名;
# 查看视图的属性信息
SHOW TABLE STATUS LIKE '视图名';
# 查看视图的定义信息
SHOW CREATE VIEW 视图名;
```



## 修改视图

**数据修改**

* 视图数据修改同表操作
* 要使视图可更新，视图中的行和基表中的行必须存在`一对一`的关系。以下情况都不支持更新：
  * 定义视图的时候指定了`ALGORITHM = TEMPTABLE`，视图将不支持INSERT和DELETE操作
  * 视图中不包含基表中所有被定义为非空又未指定默认值的列，视图将不支持INSERT操作
  * 在定义视图的SELECT语句中使用了`JOIN联合查询`，视图将不支持INSERT和DELETE操作
  * 在定义视图的SELECT语句后的字段列表中使用了`数学表达式`或`子查询`，视图将不支持INSERT，也不支持UPDATE使用了数学表达式、子查询的字段
  * 在定义视图的SELECT语句后的字段列表中使用了`DISTINCT`、`聚合函数`、`GROUP BY`、`HAVING`、`UNION`等，视图将不支持INSERT、UPDATE、DELETE
  * 在定义视图的SELECT语句中包含了子查询，而子查询中引用了FROM后面的表，视图将不支持INSERT、UPDATE、DELETE
  * 视图定义基于一个`不可更新视图`
  * 常量视图

>总的来说，视图作为虚拟表，主要用于方便查询，不建议更新数据

**视图修改**

```sql
# 方式1
CREATE OR REPLACE VIEW 视图名
AS
SELECT ...

# 方式2
ALTER VIEW 视图名
AS
SELECT ...
```



## 删除视图

```sql
DROP VIEW IF EXISTS 视图名;
```

>基于视图A、B创建了视图C，如果将A或者B删除，会导致C的查询失败。这样的视图C需要手动删除或修改，否则影响使用



## 总结

优点：

1. 简化查询，操作简单
2. 减少数据冗余
3. 数据安全
4. 适应灵活多变的需求
5. 能够分解复杂的查询逻辑

缺点：

* 如果视图过多，会导致数据库维护成本的问题：

  基表数据变更，我们需要及时对相关视图进行维护。特别是基于嵌套的视图，维护会变得比较复杂，可读性差，容易变成系统的潜在隐患。因为创建视图的SQL查询可能会对字段重命名，也可能包含复杂的逻辑，这些都会增加维护的成本



# 字符集

```sql
SHOW VARIABLES LIKE '%character%';
# character_set_server: 服务器级别的字符集
# character_set_database: 当前数据库的字符集
# character_set_client: 服务器解码请求时使用的字符集
# character_set_connection: 服务器处理请求时会把请求字符串从character_set_client转为character_set_connection
# character_set_results: 服务器向客户端返回数据时使用的字符集

# 修改数据库的字符集
ALTER DATABASE 数据库名 CHARACTER SET 'utf8';
# 修改表的字符集
ALTER TABLE 表名 CONVERT TO CHARACTER SET 'utf8';

# 查看MySQL支持的字符集
SHOW CHARSET;
SHOW CHARACTER SET;
```



## 字符集级别

* 服务器级别

  通过修改MySQL配置文件修改

* 数据库级别

  ```sql
  CREATE DATABASE 数据库名
  [CHARACTER SET 字符集]
  [COLLATE 比较规则];
  ```

* 表级别

  ```sql
  CREATE TABLE 表名
  [CHARACTER SET 字符集]
  [COLLATE 比较规则];
  ```

* 列级别

  ```sql
  CREATE TABLE 表名 (
    列名 类型 [CHARACTER SET 字符集] [COLLATE 比较规则],
    ...
  );
  ```



## utf8与utf8mb4

`utf8`字符集表示一个字符需要1~4个字节存储，但我们常用的一些字符使用1~3个字节就可以了。所以MySQL有两种`utf8`的字符集：

* `utf8mb3`：阉割的`utf8`字符集，只使用1~3个字节表示字符
* `utf8mb4`：标准的`utf8`字符集，使用1~4个字节表示字符

`utf8`是`utf8mb3`的别名



## 比较规则

指的是对字符进行比较、排序的规则

```sql
# 查看比较规则
SHOW COLLATION LIKE 'utf8%'
# 一般使用utf8mb4_general_ci
```

>https://zhuanlan.zhihu.com/p/166682278



## 请求到响应过程中字符集的变化

1. 我们知道客户端发给服务端的SQL语句，本质上就是一个字符串
2. 服务端在收到客户端的SQL语句后，会先使用系统变量character_set_client所指定的字符集对其进行解码，然后再使用系统变量character_set_connection所指定的字符集对其进行编码
3. 如果 系统变量character_set_connection所指定的字符集 与 SQL语句所指向的某列(字段)的字符集 不一致，则SQL语句还需要再次进行解码-编码操作
4. 最后，将查询结果先使用 具体的列(字段)使用的字符集 进行解码，再使用系统变量character_set_results所指定的字符集进行编码，并返回给客户端

![字符集过程](字符集过程.png)

>疑问：character_set_connection是否有必要？
>
>字符集的转换在请求过程中有两步，character_set_client->character_set_connection->列使用字符集
>
>而在响应过程中就一步，列使用字符集->character_set_results
>
>由此引发为什么不跳过character_set_connection这个问题的讨论，翻阅了一些资料，也没有找到一个能说服我的结论。
>
>有解释为 虽然字符串与列进行比较会使用列的字符集，但数据库操作并非都与表有关。
>
>如果是这样的解释的话，为什么不直接使用character_set_client指定的字符集？



## 大小写规范

* Windows下不区分大小写
* Linux下：
  * 数据库名、表名、表别名、变量名严格区分大小写
  * 关键字、函数名称在SQL中不区分大小写
  * 列名与列的别名在所有情况下都忽略大小写

>建议：
>
>1. 关键字和函数名全部大写
>
>2. 数据库名、表名、表别名、字段名、字段别名等全部小写
>
>3. SQL语句必须以分号结尾



# sql_mode

* 宽松模式
* 严格模式

```sql
select @@session.sql_mode;
select @@global.sql_mode;
# 或者
show variables like 'sql_mode';

# 临时设置设置sql_mode（永久修改sql_mode需更改配置文件）
SET GLOBAL sql_mode = '';
SET SESSION sql_mode = ''; # 多个模式用,分割
```



# 用户与权限管理

## 用户管理

MySQL用户可以分为`普通用户`和`root`用户

```sql
# 登录
mysql -h hostname|hostIP -P port -u username -p DatabaseName -e "sql语句"
```

* `-h`后面接主机名或主机IP
* `-P`后面接端口号，默认端口`3306`，不使用该参数会自动连接到`3306`端口
* `-u`后面接用户名
* `-p`会提示输入密码
* `DatabaseName`指明登录到哪个数据库
* `-e`后面可以加SQL语句。登录后立刻执行该语句，然后退出MySQL服务器

### 创建用户

```sql
# 查看当前用户
use mysql;
select host,user from user;

CREATE USER 用户名 [IDENTIFIED BY 密码];
```

用户名由`用户@host`构成，`host`默认是`%`

```sql
CREATE USER 'zhangsan'@'%' IDENTIFIED BY '123456';
```

### 修改用户

```sql
UPDATE mysql.user SET USER='li4' WHERE USER='zhangsan';
FLUSH PRIVILEGES;
```

### 删除用户

```sql
DROP USER 用户名[,用户名]; # 默认删除%下的用户，也可指定host
DROP USER 'zhangsan'@'localhost';
```

### 设置密码

* 修改自己的密码

  推荐写法：

  ```sql
  ALTER USER USER() IDENTIFIED BY '新密码';
  # 或者
  SET PASSWORD='新密码';
  ```

* 修改其他用户的密码

  ```sql
  ALTER USER 用户名 IDENTIFIED BY '新密码';
  # 或者
  SET PASSWORD FOR 用户名='新密码';
  ```

### 密码管理

1. 密码过期：要求定期修改密码
2. 密码重用限制：不允许使用旧密码
3. 密码强度评估：要求使用高强度密码



## 权限管理

### 权限列表

```sql
SHOW PRIVILEGES;
```

### 原则

1. 只授予能满足需要的最小权限
2. 创建用户时限制用户的登录主机
3. 为每个用户设置满足密码复杂度的密码
4. 定期清理不需要的用户，回收权限或删除用户

### 授予权限

授权的方式有两种，分别是通过`角色赋予用户给用户权限`和`直接给用户授权`

```sql
GRANT 权限1,权限2... ON 数据库名称.表名称 TO 用户名 [IDENTIFIED BY '密码'];
# 该操作如果没有该用户，会直接新建个用户
# 授予通过网络登录的joe用户，对所有表的全部权限，密码设为123。注意这里唯独不包括GRANT的权限
GRANT ALL PRIVILEGES ON *.* TO 'joe'@'%' IDENTIFIED BY '123';
# 如果需要赋予包括GRANT的权限，添加参数 WITH GRANT OPTION 即可
# 可以使用GRANT重复给用户添加权限，权限会叠加，而非覆盖
```

### 查看权限

```sql
# 查看当前用户权限
SHOW GRANTS;
SHOW GRANTS FOR CURRENT_USER;
SHOW GRANTS FOR CURRENT_USER();
# 查看其他用户的权限
SHOW GRANTS FOR 用户名;
```

### 回收权限

```sql
REVOKE 权限1,权限2... ON 数据库名称.表名称 FROM 用户名;
# 收回全部权限
REVOKE ALL PRIVILEGES ON *.* FROM 'joe'@'%';
```



## 权限表

MySQL通过权限表来控制用户对数据库的访问，权限表存放在`mysql`数据库中。这些权限表中最重要的是`user表`、`db表`，除此之外，还有`table_priv表`、`column_priv表`、`procs_priv表`。在MySQL启动时，这些数据库表中权限信息会读入内存。

| 表名        | 描述                 |
| ----------- | -------------------- |
| user        | 用户账户及权限信息   |
| db          | 数据库层级的权限     |
| table_priv  | 表层级的权限         |
| column_priv | 列层级的权限         |
| procs_priv  | 存储的过程和函数权限 |



## 访问控制

当MySQL允许一个用户执行各种操作时，它首先核实用户向MySQL服务器发送的连接请求，然后确认用户的操作请求是否被允许。这个过程称为MySQL中的`访问控制过程`。MySQL的访问控制分为两个阶段：

* 连接核实阶段

  当用户试图连接MySQL服务器时，服务器基于用户的身份以及用户是否提供正确的密码验证身份来确定接受或拒绝连接。即客户端用户会在连接请求中提供用户名、主机地址、用户密码，MySQL服务器接收到用户请求后，会**使用user表中的host、user和authentication_string这三个字段匹配客户端提供信息。**

  **如果连接核实没有通过，服务器就完全拒绝访问；否则，服务器接收连接，然后进入等待用户请求阶段。**

* 请求核实阶段

  在建立连接后，对此连接上进来的每个请求，服务器检查该请求要执行什么操作、是否有足够的权限来执行它，这正是需要授权表中的权限列发挥作用的地方。这些权限可以来自user、db、table_priv和column_priv表。

  确认权限时，MySQL首先`检查user表`，如果指定的权限没有在user表中授予，就会继续`检查db表`，db表是下一安全层级，其中的权限限定于数据库层级，该层级的SELECT权限允许用户查看指定数据库的所有表中数据；如果在该层级没有找到限定的权限，则继续`检查tables_priv表`以及`columns_priv表`，如果所有权限表都检查完毕，但还没有找到允许的权限操作，MySQL将返回错误信息

>提示：MySQL通过向下层级的顺序检查权限表，但并不是所有的权限都要执行该过程。



## 角色管理

### 创建角色

```sql
CREATE ROLE 'role_name'[@'host_name']
```

角色名称的命名规则和用户名类似。如果`host_name`省略，默认是`%`，`role_name`不可省略

### 分配角色权限

```sql
GRANT ALL PRIVILEGES ON *.* TO 角色名;
```

### 查看角色权限

```sql
SHOW GRANTS FOR 角色名;
# 查询当前角色
SELECT CURRENT_ROLE(); # 如果角色未激活，结果将显示NONE
```

### 回收角色权限

```sql
REVOKE ALL PRIVILEGES ON *.* FROM 角色名;
```

### 删除角色

```sql
DROP ROLE 角色名;
```

### 给用户分配角色

```sql
GRANT 角色名 TO 用户名;
```

MySQL创建角色后，默认都是没有激活的，也就是不能用，必须要`手动激活`

### 激活角色

```sql
# 方式1
SET DEFAULT ROLE ALL TO 用户名; # 用户需要重新登录
# 方式2
SET GLOBAL activate_all_roles_on_login='ON'; # 对所有角色永久激活
```

### 回收用户的角色

```sql
REVOKE 角色名 FROM 用户名;
```

### 设置强制角色

强制角色是给每个创建账户的默认角色，不需要手动设置。强制角色无法被回收或删除



# MySQL结构目录

## 主要目录结构

```shell
find / -name mysql
```

* 数据库文件存放路径：`/var/lib/mysql/`

  数据目录对应着一个系统变量`datadir`

  ```sql
  show variables like 'datadir';
  ```

* 相关命令目录：`/usr/bin`和`/usr/sbin`

* 配置文件目录：`/usr/share/mysql`和`/etc/mysql`



## 数据库和文件系统的关系

### 1.默认数据库

* `mysql`

  MySQL系统自带的核心数据库，存储了MySQL的用户信息和权限信息，一些存储过程、事件的定义信息，一些运行过程中产生的日志信息，一些帮助信息以及时区信息等

* `information_schema`

  MySQL系统自带的数据库，这个数据库保存着MySQL服务器`维护的所有其他数据库的信息`，比如有哪些表、哪些视图、哪些触发器、哪些列、哪些索引。这些信息并不是真实的用户数据，而是一些描述性信息，有时候也被称为`元数据`。在该数据库中提供了一些以`innodb_sys`开头的表，用于表示内部系统表

* `performance_schema`

  MySQL系统自带的数据库，这个数据库主要保存MySQL服务器运行过程中的一些状态信息，可以用来`监控MySQL服务的各类性能指标`。包括统计最近执行了哪些语句，在执行过程的每个阶段都花费了多长时间，内存的使用情况等信息

* `sys`

  MySQL系统自带的数据库，这个数据库主要通过`视图`的形式把`information_schema`和`performance_schema`结合起来，帮助系统管理员和开发人员监控MySQL的技术性能

### 2.数据库在文件系统中的表示

每当创建一个数据库，MySQL会帮我们做两件事：

1. 在数据目录下创建一个和数据库名同名的子目录
2. 在该子目录下创建一个`db.opt`的文件（仅限MySQL5.7及之前版本），这个文件中包含了该数据库的各种属性，比如字符集和比较规则



### 3.表在文件系统中的表示

#### InnoDB存储引擎模式

* 系统表空间

  默认情况下，InnoDB会在数据目录下创建一个`ibdata1`文件，大小为12M。这个文件就是系统表空间在文件系统上的表示。

  这是个自扩展文件，当不够用时会自己增加文件大小。

  从MySQL5.5.7到MySQL5.6.6之间的各个版本，表中的数据都会默认存储到系统表空间。

* 独立表空间

  在MySQL5.6.6之后的版本，InnoDB会为每个表建立一个独立表空间，对应文件为数据库子目录下的`.idb`文件。

  * 5.7版本数据库文件夹下每个表会有`.frm`和`.ibd`两个文件，`.frm`用于存放表结构，`.ibd`用于存放表数据和索引。
  * 8.0版本每个表仅`.ibd`一个文件。

#### MyISAM存储引擎模式

数据与索引分开存储

8.0版本中数据库文件夹下每个表会有`.sdi`、`.MYD`、`.MYI`三个文件

* `.MYD`存放表数据
* `.MYI`存放索引
* `.sdi`存放表结构（5.7版本中为`.frm`）
