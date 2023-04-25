[toc]

# SQL语句

联系学习网站:　[SQLBolt - Learn SQL - Introduction to SQL](https://sqlbolt.com/lesson/introduction)

**与数据库沟通的语言**:

- SQL(**Structured Query Language**), 结构化查询语言, 简称SQL
- 使用SQL语句就能对数据库进行操作
- 常见的关系型数据库中的SQL语句都是比较相似的

## 一. 常用规范

- 关键字使用大写的, 如: CREATE, TABLE
- 一条语句需要以`;`结尾
- 如果遇到关键字作为表名或者字段的名称时, 可以使用``包裹



## 二. SQL语句分类

- **DDL(Data Definition Language)**: 数据定义语言
  - 可以通过DDL语句对**数据库或者表**进行:**创建, 修改, 删除等操作**
- **DML(Data Manipulation Language)**: 数据操作语言
  - 可以通过DML语句对**表**进行: **添加, 删除, 修改等操作**(不包含查)
- **DQL(Data Query Language)**: 数据查询语言(重点)
  - 可以通过DQL语句从数据库中**查询记录**
- **DCL(Data Control Language)**: 数据控制语言
  - 对**数据库, 表格的权限**进行相关访问控制操作



## 三. SQL数据类型

- MySQL支持的数据类型: **数字类型, 日期和时间类型, 字符串(字符和字节)类型, 空间类型和JSON数据类型**

### 1. 数字类型

- **整数类型**

| Type      | Storage(Byte) | Min      | Max     | Max value unsigned |
| --------- | ------------- | -------- | ------- | ------------------ |
| TINYINT   | 1             | -128     | 127     | 255                |
| SMALLINT  | 2             | -32768   | 32767   | 65535              |
| MEDIUMINT | 3             | -8388608 | 8388607 |                    |
| INT       | 4             |          |         |                    |
| BIGINT    | 8             |          |         |                    |

- **浮点数字**

| Type   | storage(Byte) | Max Value Unsigned |
| ------ | ------------- | ------------------ |
| FLOAT  | 4             |                    |
| DOUBLE | 8             |                    |

- 精确数字类型
  - **DECIMAL, NUMERIC**(DECIMAL是NUMERIC的实现形式)



### 2. 日期类型

- **YEAR**以YYYY格式显示值
  - 范围1901到2155(包括0000)
- **DATE**类型用于具有日期部分但没有时间部分的值
  - DATE以格式YYYY-MM-DD显示值
  - 支持的范围是`1000-01-01 到 9999-12-31`
- **DATETIME**类型用于包含日期和时间部分的值
  - 格式:`YYYY-MM-DD hh:mm:ss`
  - 范围:`1001-01-01 00:00:00 到 9999-12-31 23:59:59`
- **TIMESTAMP**数据类型同于同事包含日期和时间部分的值
  - TIMESTAMP以格式`YYYY-MM-DD hh:mm:ss`显示值
  - 范围为UTC的时间范围: `1970-01-01 00:00:01 到 2038-01-19 03:14:07`



### 3. 字符串类型

- **CHAR**类型: 在创建表时为固定长度, 长度可以是0到255之间的任何值
  - 在被查询时, 会删除后面的空格
- **VARCHAR**类型的值是**可变长度**的字符串, 长度可以指定为0到65535之间的值
  - 在被查询时, 不会删除后面的空格
  - VARCHAR(20): 表示字符长度为0-20
- **BINARY和VARBINARY**类型用于存储二进制的字符串(先将字符转换为字节), 存储的是字节字符串
- **BLOB用于存储大的二进制文件**



### 4. 图片和音频文件

- 大的图片和音频文件一般都是将**服务器上的路径**存储到数据库中



## 四. 表约束

### 1. 主键

- 一张表中为了区分**每一条记录的唯一性**, 必须**有一个字段是永远不会重复的**, 并且不会为空(类似id), 这个字段我们通常将其设置为**主键**(可以有多个主键)
  - 主键是表中唯一的索引
  - 必须为**NOT NULL**

取值:

- UNIQUE: 唯一

- NOT NULL 不能为空
- DEFAULT 默认值
- AUTO_INCREMENT 自动递增

#### 1.1 联合主键

- 一般用于多对对关系表

```sql
CREATE TABLE IF NOT EXISTS `label`(
	PRIMARY KEY(moment_id,label_id)
)
```



### 2. 外键

- 外键约束也是常见的约束手段(多表关系)

- **删除或修改被引用的外键**(默认情况下无法修改被引用的外键)

  - **RESTRICT(默认属性)**: 当更新或删除某个记录时, 会检查该记录是否有关联的外键记录, 有的话会报错, 不允许删除或更新

  - **NO ACTION**: 和RESTRICT是一致的, 是在SQL标准中定义的

  - **CASCADE**: 当更新或者删除某个记录时, 会检查记录是否有关联的外键记录
    - 更新: 那么会更新对应的记录
    - 删除: 那么关联的记录也会被一起删除掉

  - **SET NULL**: 当更新或删除某一个记录时, 会检查记录是否有关联的外键记录, 有的话, 就将对应的值设为NULL

## 五. DDL语句

- 可以通过DDL(Data Definition Language)语句对**数据库或者表**进行:**创建, 修改, 删除等操作**

```sql
# 对数据库的操作
# 1. 查看所有的数据库
SHOW DATABASES;

# 2. 使用某一个数据库
USE music_db;

# 3. 查看当前正在使用的数据库
SELECT DATABASE();

# 4. 创建数据库
CREATE DATABASE IF NOT EXISTS test_demo;

# 5. 删除数据库
DROP DATABASE IF EXISTS  test_demo;
```

```sql
# 对表的操作
# 1. 查看当前数据库中有哪些表
SHOW TABLES;

# 2. 查看某一张表的结构
DESC t_singer;

# 3. 创建一张新的表, 如果表名为关键字则用``引起来, 并且给它的一些字段添加限制
CREATE TABLE IF NOT EXISTS `tests`(
	id INT PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(20) UNIQUE NOT NULL,
	level INT DEFAULT(0),
	telPhone VARCHAR(20) UNIQUE
)

# 4. 删除表
DROP TABLE IF EXISTS users;

# 5. 修改表
# 5.1 修改表名
ALTER TABLE tests RENAME TO tb_tests;
# 5.2 添加新的字段
ALTER TABLE tb_tests ADD time DATETIME;
# 5.3 修改字段的名称
ALTER TABLE tb_tests CHANGE time totalTime DATETIME;
# 5.4 删除某一个字段(field)
ALTER TABLE tb_tests DROP totalTime;
# 5.5 修改某一个字段的类型
ALTER TABLE tb_tests MODIFY level BIGINT;
```





## 六. DML语句

- 可以通过DML(Data Manipulation(操纵) Language)语句对**表**进行: **添加, 删除, 修改等操作**(不包含查)

```sql
# 1. DML插入数据语句
INSERT INTO `tb_products` (title,description,price,publishTime) VALUES ("白先","不是白象我不吃",199,'2020-6-7');

# 2. DML语句
# 2.1 删除全部数据语句(尽量别用,删库跑路)
DELETE FROM `tb_products`;
# 2.2 条件删除
DELETE FROM `tb_products` WHERE title = '白';

# 3. DML修改语句
# 3.1 未告知是哪一行的数据时, 默认会修改全部数据
UPDATE `tb_products` SET price = 8888;
# 3.2 提供修改条件
UPDATE `tb_products` SET price = 6666 WHERE id = 6;

# 4. 当修改某一个数据时, 触发某种改变
# 这里是为每条数据设置一个初始值, 且当这条数据发生更新时, 这个字段也会发生更新
ALTER TABLE `tb_products` ADD updateTime TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
```



## 七. DQL语句

- **DQL(Data Query Language)**: 数据查询语言(重点)
  - 可以通过DQL语句从数据库中**查询记录**
  - SELECT用于从一个或者多个表中检索选中的行(record)



- **查询格式**

```sql
# 查询语句的格式
SELECT select expr [,select expr]...
			[FROM table_references]
			[WHERE where_condition]
			[ORDER BY expr [ASC | DESC]]
			[LIMIT {[offset,] row_count | row_count OFFSET offset}]
			[GROUP BY expr]
			[HAVING where_condition]
```

- **简单的筛选查询结果**

```sql
# 1. 查询所有的数据
SELECT * FROM `tb_products`

# 2. 查询所有数据, 但只显示指定的数据, 也可以起别名(避免多表重复), AS关键字可以省略
SELECT id AS phoneId, brand, title, price FROM `products`;

# 3. 筛选查询出来的数据
SELECT * FROM `tb_products` WHERE price < 1000;
# 3.1 逻辑运算符
SELECT * FROM `tb_products` WHERE brand = '华为' && price > 2000;
SELECT * FROM `tb_products` WHERE brand = '华为' AND price > 2000;
SELECT * FROM `tb_products` WHERE brand = '华为' || price > 2000;
SELECT * FROM `tb_products` WHERE brand = '华为' OR price > 2000;
# 3.2 区间写法
SELECT * FROM `tb_products` WHERE price BETWEEN 1000 AND 2000;

# 3.3 枚举多个结果的其中之一(等同于or)
SELECT * FROM `tb_products` WHERE brand IN ('华为','小米');

# 4 模糊查询
# 模糊查询使用LIKE关键字, 结合两个特殊符号, %表示匹配任意个的任意字符, _表示匹配一个的任意字符
SELECT * FROM `tb_products` WHERE title LIKE '华_'; # 匹配到华为
```

- **升序和降序**

```sql
# 给查询结果排序, ASC: 升序排列, DESC: 降序排列
# 降序
SELECT * FROM `tb_products` WHERE title LIKE '华_' ORDER BY score DESC;
# 升序
SELECT * FROM `tb_products` WHERE title LIKE '华_' ORDER BY score ASC;
```

- **分页查询**
  - 不需要`WHERE`进行筛选了

```sql
# 5. 分页查询
# 5.1 默认从0开始查起
SELECT * FROM `tb_products` LIMIT 1;
# 5.2 指定偏移多少条数据, 即从第几条开始查
SELECT * FROM `tb_products` LIMIT 1 OFFSET 2;
# 5.3 另一种写法, 效果与上面一样
SELECT * FROM `tb_products` LIMIT 2,1;
```



## 八. 聚合函数

- 聚合函数: **对查询到的值的集合进行操作的组(集合)函数**
- MySQL中内置了一些**进行计算的函数**
  - `AVG()`: 拿到传入的集合的平均值
  - `MAX()`: 拿到传入的集合中的最大值
  - `MIN():` 拿到传入的集合中的最小值
  - `SUM()`: 计算传入集合的总和
  - `COUNT()`: 计算传入的集合中元素的个数, 类似`arr.length`, 也可以计算筛选出来的数据个数`COUNT(*)`
  - `JSON_OBJECT()`: 将传入的属性转换为一个对象

```sql
# 这里拿到的price是, 所有符合条件的record的price数据组成的集合
SELECT price FROM `products` WHERE title = 	'华为';

# 1. 计算平均价格
SELECT AVG(price) FROM `products` WHERE title = '华为';
```



## 九. Group By(HAVING)

- 聚合函数将所有拿到的数据默认分为了一组, **如果我们想要将拿到的数据划分为多个组, 就可以使用GROUP BY**

- GROUP BY通常和聚合函数一起使用
  - 相同数据进行分组, 再对每一组数据进行聚合函数的计算
- **使用GROUP BY进行分组后的数据无法使用WHERE进行筛选, 需要使用HAVING**

```sql
# 1. 根据brand分组, 取出分完组的数据中的brand,max(price),min(price)
# 先分组再从分组中取数据
SELECT brand, MAX(price), MIN(price) from 'products' GROUP BY brand

# 2. 使用HAVING进行筛选数据
SELECT MAX(price) maxPrice from 'products' GROUP BY brand HAVING maxPrice > 2000;
```

- 分组后的每组数据==只能显示一行==, 因此所有除分组**依赖的数据**之外, 所有的数据都**需要进行整合才能显示**

​	因为不可能在一个字段内显示所有的不同的数据, 同时也不能确保所有集合内的数据中某一个字段是整体相同的

**但如果在查询列表中有id(PRIMARY KEY)能保证所有数据来自另一张表的同一行数据, 这些数据就能进行整合![Snipaste_2023-01-13_13-32-38](.\图片\Snipaste_2023-01-13_13-32-38.png)**

- 如上面这组数据显示, 当以brand进行分组的时候, 不能单独展示任何一列数据(报错), 因为不能保证这一组数据都是相同的
- 但是如果有主键(Primary key)的保证, 则能够显示数据



## 十. 子查询

- 在一个查询中, 包含另一个查询, 将另一个查询出来的结果作为一个字段
- 可以用子查询避免多表之间的相互影响
  - LEFT JOIN 时可能产生重复数据

```SQL
# 这里是在查询A表时, 统计在B表中对该评论的子评论数
SELECT
	m.id id, m.content content
	JSON.OBJECT('id',u.id) user
	
	(SELECT COUNT(*) FROM `comment` WHERE comment.moment_id = m.id) commentCount
FROM `moment` m
LEFT JOIN user u ON u.id = m.user_id
LIMIT 10 OFFSET 0
```



## 十一. 高难度SQL

```sql
# 动态表moment,评论表comment和标签表label, 同时还有动态和标签关系表moment_label
SELECT 
	m.id id, m.content content,
	JSON_OBJECT('id',u.id,'name',u.name) user,
	(
    	SELECT
        	jJSON_ARRAYAGG(JSON_OBJECT(
            	'id',c.id, 'content',c.content,'contentId',c.comment_id,
                'user',JSON_OBJECT('id',cu.id,'name',cu.name)
            ))
        FROM comment c
        LEFT JOIN user cu ON c.user_id = cu.id
        WHERE c.moment_id = m.id
    ) comments,
    (
    	JSON_ARRAYAGG(JSON_OBJECT(
        	'id',l.id,'name',l.name
        ))
    ) labels
FROM moment m
LEFT JOIN user u ON u.id = m.user_id
LEFT JOIN moment_label ml ON ml.moment_id = m.id
LEFT JOIN label l ON ml.label_id = l.id
WHERE m.id = 2
GROUP BY 
```

