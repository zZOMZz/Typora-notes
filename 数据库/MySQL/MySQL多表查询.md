[toc]

# MySQL多表查询

## 一. 外键约束

- 一张表的内容约束着另外一张表的取值范围
- 用于多表之间建立约束取值关系

```sql
# 1. 创建表时添加外键
CREATE TABLE IF NOT EXISTS `t_songs`(
	id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20),
    duration INT DEFAULT 0,
    singer VARCHAR(10),
    brand_id INT,
    FOREIGN KEY(brand_id) REFERENCES brand(id) 
)
# 2. 修改表时添加外键
ALTER TABLE `products` ADD 'brand_id' INT;
ALTER TABLE `products` ADD FOREIGN KEY(brand_id) REFERENCE brand(id);

# 3. 再更新数据
UPDATE `products` SET `brand_id` = 1 WHERE `brand` = '华为';
```

- **删除或修改被引用的外键**(默认情况下无法修改被引用的外键)
  - **RESTRICT(默认属性)**: 当更新或删除某个记录时, 会检查该记录是否有关联的外键记录, 有的话会报错, 不允许删除或更新
  - **NO ACTION**: 和RESTRICT是一致的, 是在SQL标准中定义的
  - **CASCADE**: 当更新或者删除某个记录时, 会检查记录是否有关联的外键记录
    - 更新: 那么会更新对应的记录
    - 删除: 那么关联的记录也会被一起删除掉
  - **SET NULL**: 当更新或删除某一个记录时, 会检查记录是否有关联的外键记录, 有的话, 就将对应的值设为NULL

```SQL
# 展示创建product的SQL语句, 拿到其中的外键名称
SHOW CREATE TABLE `product`
# 删除一个外键
ALTER TABLE `product` DROP FOREIGN KEY [外键名称];
# 添加一个可以跟随更新和删除的外键
ALTER TABLE `product` ADD FOREIGN KEY(brand_id) REFERENCES brand(id)
					  ON UPDATE CASCADE
					  ON DELETE CASCADE;
```



## 二. 多表查询

- 在多张表格之间**添加引用关系, 有效减少冗余数据的产生**(可以利用独一无二的id进行匹配)



- 默认多表查询是直接**笛卡尔乘积**也称为**直积**, 表示为`X * Y`
  - 第一张表中的每个数据都会和第二张表的每条数据进行结合
  - **可以对所有数据进行过滤(两表外键相同), 但也不是最好的方法**

### 1. SQL JOIN

- **左连接**: **获取到的数据以左边的表为主, 左边的数据能全部展示**
- 右连接
- **内连接**:
  - 上面的写法在两张表连接时就会建立约束关系, 再决定查询后的结果
  - 下面的写法是先查询出所有的数据, 再进行筛选
- 全连接: 两张表的数据全部展示(MySQL不支持, 需要使用UNION)

```sql
# 1. 左连接, 查询所有数据
SELECT * FROM `products` LEFT JOIN `brands` ON `products`.brand_id = brands.id;
# 2. 左连接, 增加条件判断, 筛选没有连接的数据
SELECT * FROM `products` LEFT JOIN `brands` ON `products`.brand_id = brands.id WHERE 
											brand_id = NULL;
											
# 3. 内连接
SELECT * FROM products INNER JOIN brand ON products.brand_id = brands.id
# 上面这种写法类似
SELECT * FROM products , brands WHERE products.brand_id = brand.id

# 4 .全连接
(SELECT * FROM `products` LEFT JOIN `brands` ON `products`.brand_id = brands.id;)
UNION
(SELECT * FROM `products` RIGHT JOIN `brands` ON `products`.brand_id = brands.id;)
```

### 2. 多对多查询

- 多对多的关系, 会建立一张专门的关系表

```sql
# 1. 创建关系表
CREATE TABLE IF NOT EXIST `student_select_course`(
	id INT PRIMARY KEY AUTO_INCREMENT,
    student_id	INT NOT NULL,
    course_id INT NOT NULL,
    FOREIGN KEY(student_id) REFERENCES students.id
    						ON UPDATE CASCADE
    						ON DELETE CASCADE,
    FOREIGN KEY(course_id) REFERENCES courses.id
    					   ON UPDATE CASCADE
    					   ON DELETE CASCADE,
)
# 2. 展示所有学生的选课情况
# 利用left join 等连接符
SELECT 
stu.name stuName, cs.name courseName, cs.price csPrice
FROM students stu
JOIN stUdent_select_course ssc ON stu.id = ssc.student_id
JOIN course cs ON ssc.course_id = cs.id
WHERE stu.name = "zzt"
```



### 3. 组织多表查询返回数据结构

- **可以将多表查询返回的结果塞入单独的属性中, 避免所有数据都在统一的对象中, 这样可以使数据结构更加清晰**

- 利用**聚合函数**

```sql
# 这样筛选出来的数据就能包含一个brand数据, brand数据包含表二中的选择的数据(object)
SELECT 
	products.id id, products.title title , products.price price
	JSON_OBJECT('id',brands.id,'name',brands.name,'website',brands.website) as brand
from products
LEFT JOIN brands
ON products.brand_id = brands.id
```

```sql
# 形成列表
SELECT
	stu.id id, stu.name name, stu.age age,
	JSON_ARRAYAGG(JSON_OBJECT("id",cs.id,"name",cs.name,"price",cs.price)) courses
FROM students stu
LEFT JOIN student_select_course ssc ON stu.id = ssc.student_id
LEFT JOIN courses cs ON cs.id = ssc.course_id
WHERE cs.id IS NOT NULL
GROUP BY stu.id
```

![Snipaste_2023-01-13_13-30-28](.\图片\Snipaste_2023-01-13_13-30-28.png)
