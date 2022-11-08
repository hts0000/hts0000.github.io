# MySQL必知必会


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

## 检索
```sql
# 查找并去重，如果有多列，则要一整行完全一样才会去重
SELECT DISTINCT prod_name FROM prod_table;

# limit限制输出的行数，第一个参数指示从第几行开始(从0开始算)，后面参数指示输出几行
SELECT prod_name FROM prod_table LIMIT 5, 3;
SELECT prod_name FROM prod_table LIMIT 3 OFFSET 5; # 一样的意思

# 根据当前列进行排序
SELECT prod_name FROM prod_table ORDER BY prod_name;
# 根据其他列排序当前列
SELECT prod_name FROM prod_table ORDER BY id;
# 根据多列进行排序，先根据prod_name列排序，再根据id列排序（只有prod_name列的值重复时，才会使用id列排序）
SELECT prod_name FROM prod_table ORDER BY prod_name, id;
# 倒序排序，想根据多个列进行降序排序，必须在每个列后面都跟上DESC
SELECT prod_name FROM prod_table ORDER BY prod_name DESC, id DESC;
```

## 过滤数据
```sql
# 过滤price=2.2的列，支持的操作符有：=, !=, <, >, <=, >=, BETWEEN
SELECT price FROM prices WHERE price=2.2;
# BETWEEN过滤，范围[2.2 ~ 3.2]
SELECT price FROM prices WHERE price BETWEEN 2.2 AND 3.2;
# 过滤非空值
SELECT prod_name FROM prod_table WHERE prod_name IS NOT NULL;

# 多过滤条件组合，OR, AND, NOT, IN，符号优先级AND > OR，可以加上括号()来明确优先级
SELECT prod_name FROM prod_table WHERE (prod_name = "apple" OR prod_name = "mongo") AND prod_price > 2.2 AND prod_price < 4.4;
# IN操作符，过滤条件是否在集合中
SELECT prod_name FROM prod_table WHERE prod_name IN ("apple", "mongo");

# 模糊匹配，%匹配多个任意字符，_匹配任意单个字符
# 模糊匹配性能很差
SELECT prod_name FROM prod_table WHERE prod_name LIKE "%apple%";
SELECT prod_name FROM prod_table WHERE prod_name LIKE "_ppl_";
```

## 正则匹配REGEXP
```sql
SELECT prod_id, prod_name FROM products WHERE prod_id REGEXP ''
```

一些特殊字符：`-`,`|`,`[]`,

![](https://raw.githubusercontent.com/hts0000/images/main/202211031517546.png)

![](https://raw.githubusercontent.com/hts0000/images/main/202211031516938.png)

![](https://raw.githubusercontent.com/hts0000/images/main/202211031516247.png)

![](https://raw.githubusercontent.com/hts0000/images/main/202211031553945.png)

`^`的双重用途
- 当在集合中`[^]`，表示否定集合的条件
- 当在集合外`^[]`，表示以该集合条件开头

MYSQL中快速测试正则表达式是否可以正确工作：`SELECT 'hts_0000@sina.com' REGEXP '^[:alnum:]{4,15}@[sina|qq|163]\\.[com|cn]$'`,

## 拼接字段
```sql
# 把输出内容格式化，有点像printf
SELECT Concat(prod_name, "(", prod_price, ")") FROM prod_table;

# 去除字段左右多余空格，RTrim去除右边空格，LTrim去除左边空格，Trim去除左右空格
SELECT Concat(RTrim(prod_name), "|", LTrim(prod_name), "|", Trim(prod_name)) FROM prod_table;

# 计算price*num的值，并将计算结果展示为expanded_price列
SELECT price, num, price*num AS expanded_price FROM prices;
```

![](https://raw.githubusercontent.com/hts0000/images/main/202211031619081.png)

MYSQL中快速测试函数调用和计算表达式的值：`SELECT 2*3;`或`SELECT Trim(' hello world ');`

## 函数
### 文本处理函数
![](https://cdn.jsdelivr.net/gh/hts0000/images/202203011745159.png)

```sql
RTrim, LTrim, Trim

# Upper, Lower
SELECT price_name, Upper(price_name) as upper_price_name, Lower(price_name) as lower_price_name FROM prices;

# Length, Locate, Soundex, SubString
```

### 时间函数
![](https://cdn.jsdelivr.net/gh/hts0000/images/202203011758459.png)

MYSQL中存储时间的数据类型为`datatime`，其会将时间存储为`YYYY-MM-DD HH:mm::ss`，因此下面的时间匹配是不可靠的，因为它只匹配了日期，而没有时间。可靠的做法是，用`Date()`函数，提取`order_date`这一列数组中的日期部分，再做匹配

```sql
# 不可靠的做法
SELECT cust_id, order_num FROM orders WHERE order_date = '2020-01-01';

# 可靠的做法
SELECT cust_id, order_num FROM orders WHERE Date(order_date) = '2020-01-01';
```

```sql
# 匹配2005年9月的所有数据
SELECT cust_id, order_num FROM orders WHERE Year(order_date) = 2005 AND Month(order_date) = 9;
```

### 数值处理函数
![](https://cdn.jsdelivr.net/gh/hts0000/images/202203011758636.png)

### 聚合函数
汇总数据，而不是检索出来。常用的汇总有：
- 汇总行数
- 行数据组之和
- 列数据之和、最大值、最小值、平均值等

![](https://cdn.jsdelivr.net/gh/hts0000/images/202203011759430.png)

```sql
# AVG, COUNT, MAX, MIN, SUM
SELECT MAX(price), MIN(price), SUM(price), COUNT(price), AVG(price) FROM prices;
```

`MAX()`一般用来找出数值或时间类型的最大值，但它也能用来找文本类型的最大值。`MIN()`它也能用来找文本类型的最小值

### 系统函数
返回系统信息，如登录用户信息、系统版本等

## 聚合数据

## 分组数据
分组允许把数据分为多个逻辑组，以便对每个组做聚合计算。

```sql
# 根据vend_id创建分组，再对每一组做聚合计算(这里的聚合是COUNT(*))
SELECT vend_id, COUNT(*) AS num_prods FROM products GROUP BY vend_id;
```

### GROUP BY 创建分组
```sql
# 根据id把ids表排序分组，COUNT(*)是对每个分组的数据进行统计
SELECT id, count(*) AS num_id FROM ids GROUP BY id;
```

### HAVING 过滤分组
`HAVING`功能跟`WHERE`类似，HAVING仅仅是因为分组查询出来的数据WHERE不能用过滤分组，仅此而已。HAVING基本上能代替WHERE所有功能。

注意，用HAVING过滤的时候，不能写别名，必须用前面聚合的表达式。如下面的例子就必须用COUNT(*)，而不能用num_id
```sql
# 分组的过滤不能使用WHERE，必须使用HAVING
SELECT id, COUNT(*) AS num_id FROM ids GROUP BY id HAVING COUNT(*) > 2;

# HAVING和WHERE合用的例子
# 返回有2个物品价格>=10的供应商id
SELECT vend_id, COUNT(*) AS num_id FROM products WHERE prod_price >= 10 GROUP BY vend_id HAVING COUNT(*) >= 2;
```

## 子查询
就是嵌套查询
```sql
# 先查询购买了TNT2的所有订单号，再根据订单号过滤出客户id
SELECT cust_id FROM orders WHERE order_num IN (SELECT order_num FROM orderitems WHERE prod_id = "TNT2");

# 为计算列使用子查询，将customers.cust_id列与子查询中的orders.cust_id列一一比较，n*m的嵌套比较关系
SELECT
	cust_id,
	cust_name,
	cust_state,
	(SELECT count(*) FROM orders WHERE orders.cust_id = customers.cust_id) AS orders
FROM customers
ORDER BY cust_name;
```

## 联结表 JOIN
联结表时，实际上就是把第一个表中的每一行与第二个表的每一行配对。相当于做了一个笛卡尔积。

内联结——基于两个表之间的相等测试。

```sql
# 将vendors表和products表联结起来，配对vendors和products表的每一行数据，当id对应时，就输出供应商名称，产品名称和产品价格，通过这种方法可以找到产品对应的供应商。
# 这种方法需要表与表之间有关系，主键和外键
SELECT vend_name, prod_name, prod_price FROM vendors, products WHERE vendors.vend_id = products.vend_id;

# 内联结写法
SELECT vend_name, prod_name, prod_price FROM vendors INNER JOIN products ON vendors.vend_id = products.vend_id ORDER BY vend_name, prod_name;
```

## 高级联结
表可以声明别名，用AS关键字。表的别名能有效缩短SQL语句。

### 自联结
表a联结表a，自己联结自己。使用场景是，我想要在当前表中找到生产了'DTNTR'这个产品ID的供应商生产的所有产品。

自联结需要用到表别名
```sql
SELECT p1.prod_id, p1.prod_name FROM products AS p1, products AS p2 WHERE p1.vend_id = p2.vend_id AND p2.prod_id = 'DTNTR';
```

自然联结

### 外部联结

内联结  
只列出ON匹配的行

左联结  
选择LEFT OUTER JOIN子句左边的所有行去匹配右边的所有行，如果右边有不匹配的行，就是NULL，ON后面的条件不匹配也会列出来

右联结  
选择RIGHT OUTER JOIN子句右边的所有行去匹配左边的所有行，如果左边有不匹配的行，就是NULL，ON后面的条件不匹配也会列出来

左右联结可以互换使用，只需要调整WHERE或FROM子句中表的顺序。

```sql
# 检索所有客户及他们的订单，包括没有订单的客户，如果用内联结，就无法检索出没有订单的客户了
SELECT c.cust_id, o.order_num FROM customers AS c LEFT OUTER JOIN orders AS o ON c.cust_id = o.cust_id;
```

### 使用带聚集函数的联结


## 组合查询
用UNION操作将多条SELECT语句组合成一个结果集。

UNION操作与多个WHERE条件查询结果完成的工作一样。但是执行性能不一样，需要试试才知道哪个性能好。

UNION并上的SELECT必须有相同的列，列的顺序可以不同。

UNION会自动去重，与WHERE结果一样。如果不想去重，用UNION ALL，UNION ALL与WHERE就不一样了，它能做到WHERE做不到的事。

```sql
# 查找产品价格小于等于5的产品，并上vend_id为1001或1002的
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price <= 5 UNION SELECT vend_id, prod_id, prod_price FROM products WHERE vend_id IN (1001, 1002);

# 与上面等价的WHERE
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price <= 5 OR vend_id IN (1001, 1002);
```

## 全文本搜索

## 插入
```sql
# 依赖表定义定义的次寻，每个值必须与表中列的定义对应上，如果定义允许为空，才能使用NULL
# 主键都是填NULL，让MYSQL自动生成
INSERT INTO Customers VALUES(NULL, 'PeP E. La', '100');

# 也可以指定列名，这样的好处是，表结构发生变化该SQL也能工作，因为它不强依赖列的次寻
# 不必总是填上所有列，没有指定的列，会被填上NULL
INSERT INTO Customers(cust_name, cust_contace, cust_email) VALUES('PeP E. La', NULL, 'pepela@gla.com');

# 插入多行
INSERT INTO Customers(
	cust_name,
	cust_contace,
	cust_email)
VALUES(
		'PeP E. La',
		NULL,
		'pepela@gla.com'
	),
	(
		'PeP E. La',
		NULL,
		'pepela@gla.com'
);
```

## 更新
```sql
UPDATE customers SET cust_email = 'test@google.com' WHERE cust_id = 10005;
```

## 删除
```sql
DELETE FROM customers WHERE cust_id = 10005;

TRUNCATE TABLE;
```

## 创建表
```sql
CREATE TABLE `test` (
  `cust_id` int(11) NOT NULL AUTO_INCREMENT,
  `cust_name` char(50) NOT NULL,
  `cust_address` char(50) DEFAULT NULL,
  `cust_city` char(50) DEFAULT NULL,
  `cust_state` char(5) DEFAULT NULL,
  `cust_zip` char(10) DEFAULT NULL,
  `cust_country` char(50) DEFAULT NULL,
  `cust_contact` char(50) DEFAULT NULL,
  `cust_email` char(255) DEFAULT NULL,
  PRIMARY KEY (`cust_id`)
) ENGINE=InnoDB AUTO_INCREMENT=10006 DEFAULT CHARSET=utf8mb4;
```

## 更新表
更新表的定义

```sql
# 增加列
ALTER TABLE Customers ADD describes CHAR(50);

# 删除列
ALTER TABLE Customers DROP COLUMN describes;
```

## 删除表
```sql
DROP TABLE customers;
```

## 重命名表
```sql
RENAME TABLE old_table TO new_table;
```

## 视图
视图是虚拟的表，虚拟表不包含数据，它包含的是一个SQL查询。

为什么要使用视图：
- 重用SQL语句
- 简化SQL操作
- 使用表的组成部分，而不是整个表
- 保护数据，可以只给到用户视图，而不是整个表

视图创建后，可以像使用表一样使用它，包括SELECT、过滤、排序、JOIN其他表或视图，甚至添加和更新数据。

视图仅仅是用来查看存储在别处的数据的一种设施，视图本身不包含数据，因此它返回的数据是从其他表中检索出来的。

视图最重要的功能就是，将一些常用的、重复的基础查询虚拟成一张表，我们可以在建视图的时候就格式化好里面的数据，使其更有用，减少重复劳动。

### 使用视图
```sql
# 创建视图
CREATE VIEW productcustomers AS SELECT cust_name, cust_contact, prod_id FROM customers, orders, orderitems WHERE customers.cust_id = orders.cust_id AND orderitems.order_num = orders.order_num;

# 查看创建视图的语句
SHOW CREATE VIEW viewname;

# 删除视图
DELETE VIEW viewname;

# 用视图重新格式化检索出来的数据
CREATE VIEW vendorlocation AS SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')') AS vend_title FROM vendors ORDER BY vend_name;
```

## 存储过程
存储过程简单来说是一条或多条SQL语句的集合，可将其视为批处理文件，但不仅限于批处理。本质就是一个函数。

存储过程简单、安全并且性能高。

存储过程创建之后，会一直存在，直至被显式删除。

存储过程不易于版本管理和调试，阿里开发规范中不建议使用存储过程。

但是存储过程可以封装数据处理、提高性能、减少数据传输带宽，具体使用要视情况而定。

### 执行/调用存储过程
```sql
# 调用productpricing这个存储过程，@pricelow表示用pricelow这个变量来存储结果，MySQL中的变量必须以@开头
CALL productpricing(@pricelow,
					@pricehight,
					@priceaverage);

# 如果是要传入变量给存储过程
CALL productpricing(1000,
					@pricelow,
					@pricehight,
					@priceaverage);
```

### 创建存储过程
在MySQL命令行中创建存储过程，需要把语句分隔符`;`改掉，不然会报语法错误。在其他数据库工具中应该不需要这样改。

一般来说，存储过程不显示结果，而是把结果返回给你指定的变量。

```sql
# 修改分隔符为//
DELIMITER //

# 创建一个名为productpricing的无参数存储过程
CREATE PROCEDURE productpricing() BEGIN SELECT Avg(prod_price) AS priceaverage FROM products; END//

# 有返回参数的存储过程
# BEGIN上面的这一块是定义入参和出参的地方
# BEGIN和END中间这一段是具体逻辑
CREATE PROCEDURE productpricing(
    OUT pl DECIMAL(8, 2), # 返回pl给调用者
    OUT ph DECIMAL(8, 2),
    OUT pa DECIMAL(8, 2)
)
BEGIN
    SELECT Min(prod_price)
    INTO pl # 表示将结果传递给pl变量
    FROM products;
    SELECT Max(prod_price)
    INTO ph
    FROM products;
    SELECT Avg(prod_price)
    INTO pa
    FROM products;
END//

# 调用上面的存储过程
CALL productpricing(@pl, @ph, @pa)//

# 使用上面的存储过程，不会产生输出，而是会返回pl、ph、pa变量，访问变量来获取结果
SELECT @pl as pricelow//
SELECT @pl as pricelow, @ph as pricehight, @pa as priceaverage//

# 定义一个具有入参的存储过程
CREATE PROCEDURE ordertotal(
    IN onumber INT, # 接受一个INT型的参数
    OUT ototal DECIMAL(8, 2)
)
BEGIN
    SELECT Sum(item_price*quantity)
    FROM orderitems
    WHERE order_num = onumber # 在这里用上了onumber
    INTO ototal;
END//

# 调用上面的存储过程
CALL ordertotal(20005, @total)//

# 修改分隔符为;
DELIMITER ;
```

### 删除存储过程
```sql
DROP PROCEDUER IF EXISTS productpricing;
```

### 更高级更复杂的存储过程
下面这个存储过程功能为，给出是否需要加税`taxable`，计算给定客户编号`onumber`的价格总计`ototal`。
```sql
-- 这些都是注释
-- Name: ordertotal
-- Parameters: onumber = order number
--             taxable = 0 if not taxable, 1 if taxable
--             ototal  = order total variable

CREATE PROCEDURE ordertotal(
    IN onumber INT,
    IN taxable BOOLEAN, # 传入的是一个布尔值
    OUT ototal DECIMAL(8, 2)
) COMMENT '订单收益总计，是否附加税' # 这条也是注释，用SHOW PROCEDURE STATUS可以看到
BEGIN

    -- Declare variable for total
    DECLARE total DECIMAL(8, 2); # 这里定义的就是局部变量
    -- Declare tax percentage
    DECIMAL taxrate INT DEFAULT 6; # 定义税率

    -- Get the order total
    SELECT Sum(item_price*quantity)
    FROM orderitems
    WHERE ordr_num = onumber
    INTO total;

    -- Is this taxable?
    IF taxable THE # 传入的布尔值为真，就加上税率
        -- Yes, so add taxrate to the total
        SELECT total+(total/100*taxrate) INTO total;
    END IF;

    -- And finally, save to out variable
    SELECT total INTO ototal; # 将最终计算结果传出去

END;
```

## 游标
从一组结果中获取数据的指针，可以任取结果集中前几行后几行和某几行，像一个迭代器。

MySQL中的游标只能在存储过程或函数中使用。

游标只能定义在存储过程和函数中，定义好之后还要显式的打开和关闭它，以启用和释放游标占用的资源。

```sql
CREATE PROCEDURE processorders()
BEGIN

	-- Declare local variables
	DECLARE o INT;

	-- Declare the cursor
	-- 从SELECT查询出来的一组数据集中定义一个游标
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;

	-- Declare continue handler
	-- 当出现02000错误时，done被设置为1
	-- 02000错误会发生在REPEAT无法提供更多循环时
	DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;

	-- Open the cursor
	-- 显式的打开游标
	OPEN ordernumbers;

	-- Get order number
	-- 从游标中获取一个数据
	FETCH ordernumers INTO o;

	-- Loop through all rows
	REPEAT
		-- Get order number
		FETCH ordernumers INTO o;
	-- End of loop
	-- 当没有更多数据可迭代了，会发生02000错误，done被置为1
	-- UNTIL 意思为直到 done 为真了，才停止
	UNTIL done END REPEAT;

	--Close the cursor
	-- 显式的关闭游标
	CLOSE ordernumbers;
END//
```

## 触发器
在`DELTE`、`UPDATE`、`INSERT`操作执行之后触发一系列操作。

## 事务
MySQL常用的引擎中有InnoDB引擎是支持事务的，MyISAM则不支持。

事务，用来保证一批SQL操作的原子性，要么全部成功，要么全部失败。

如果事务中某条SQL语句执行出错了，那么整个事务会被自动回滚。

当事务中执行了ROLLBACK或COMMIT，事务会自动关闭。

MySQL命令行中，所有操作都是默认自动提交的，执行的DELETE、INSERT等操作，本来应该是需要显式的COMMIT才会修改，但是因为设置了默认提交，所以会立即生效，如果想改变默认提交行为，可以使用下面的命令。
```sql
# 默认不提交
SET autocommit=0;
# 默认提交
SET autocommit=1;
```

事务处理的术语：
- 事务(transaction) 指一批SQL语句
- 回退(rollback) 回退事务，撤销未commit前的所有事务操作，回退不能回退CREATE和DROP操作
- 提交(commit) 将事务执行结果写入数据库表
- 保留点(savepoint) 设置一个储存点，回退到这个储存点，而不是整个事务

```sql
# 开始事务
START TRANSACTION
SELECT * FROM orders;
DELETE FROM orders; # 删除orders表中的所有行
SELECT * FROM orders;
# 回退这个事务，其实就是回退了所有改变数据的操作，上面只有DELETE，回退SELECT是没有意义的
# 执行完整ROLLBACK之后事务就隐形关闭了
ROLLBACK; 

# 需要重新开启事务
START TRANSACTION
SELECT * FROM orders;
DELETE FROM orders;
# 设置一个保留点，可以ROLLBACK到这个位置，上面的操作不会被回滚
SAVEPOINT delete1;
ROLLBACK TO delete1;
```

## 字符集管理
两个概念，字符集(CHARACTER)和其对应的校对(COLLATION)。

字符集负责字符串的编码规则，常见的有 utf8 和 utf8mb4。

utf8 和 utf8mb4 的区别：
- utf8是变长字节编码，会用1~4字节去编码，MySQL中最多使用3字节，因此无法支持emoji表情等需要4字节才能表示的字符
- utf8mb4是固定长度的编码，统一使用4字节，完全涵盖世上所有字符

校对是对应的字符集的排序字符集。字符串除了存储还需要排序和比较，校对就是干这件事的。

校对字符集也有两类常见的，utf8mb4_general_ci 和 utf8mb4_unicode_ci。

utf8mb4_general_ci 和 utf8mb4_unicode_ci 的区别：
- utf8mb4_general_ci没有实现基于标准Unicode编码的排序，对于一些特殊的字符和emoji排序的结果可能不是期望值
- utf8mb4_unicode_ci实现了基于标准Unicode编码的排序，能够精确的排序所有字符，但是相对更加耗时

MySQL8.0中默认字符集是utf8mb4，默认校对集是utf8mb4_0900_ai_ci。  
在8.0之前，应该选择utf8mb4和utf8mb4_unicode_ci或者是utf8_unicode_520_ci如果有的话。

```sql
# 查看所有字符集及其默认校对
SHOW CHARACTER SET;
# 查看所有可用校对
SHOW COLLATION;

# 查看创建数据库或表时使用的字符集和校对
SHOW VARIABLES LIKE 'character%';
SHOW VARIABLES LIKE 'collation%';

# MySQL允许对数据库、表和每一列单独设置
# 单独设置数据库
CREATE DATABASE test DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE TABLE test_table(
	id INT,
	name VARCHAR(20),
	email VARCHAR(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci # 单独设置列
)
DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci; # 单独设置表
```

## 安全管理
### 用户管理
MySQL用户账户信息存储在mysql.user表中。

有三种方法可以创建用户：
- CREATE语句
- GRANT语句，MySQL8.0中不在支持
- 在mysql.user表中INSERT一条记录

CREATE语句创建用户的语义是最清晰的。GRANT语句是用来给用户赋予权限的。直接INSERT则是强烈不建议使用。

```sql
# 查看所有用户信息
SELECT * FROM mysql.user;
# 创建一个用户，基于给定的密码
CREATE USER xiaoming IDENTIFIED BY 'p@$$w0rd';
# 重命名用户，仅支持5.0之后的版本，5.0之前使用UPDATE更新mysql.user表
RENAME USER xiaoming TO xiaoli;
# 更改用户的密码，不指定用户名时，更新当前用户的密码
SET PASSWORD FOR xiaoli = Password('n3e p@$$w0rd');
# 删除用户
DROP USER xiaoli;
```

### 用户权限管理
管理用户权限，使用GRANT语句，GRANT语句要求至少提供如下信息：
- 要授予的权限
- 授予访问权限的库或表
- 要授予权限的用户名

GRANT的反操作的REVOKE，用来撤销权限。

```sql
# 新建用户没有访问权限，能登录数据库，但看不到任何数据，也不能操作任何数据
# 查看用户的权限
SHOW GRANTS FOR xiaoli;

# 输出结果为
# +------------------------------------+
# | Grants for xiaoli@%                |
# +------------------------------------+
# | GRANT USAGE ON *.* TO 'xiaoli'@'%' |
# +------------------------------------+

# 其中USAGE ON *.* 表示该用户对任何库和表没有任何权限(USAGE表示没有权限)
# 'xiaoli'@'%' 表示 xiaoli用户可以从任意一个ip登录上来，'%'匹配任意ip或主机名

# 给xiaoli添加对orders表的查询和插入权限
GRANT SELECT, INSERT ON crashcourse.orders TO xiaoli;
# 撤销xiaoli的插入权限
REVOKE INSERT ON crashcourse.orders TO xiaoli;
```

可以被授予或撤销的权限

![](https://raw.githubusercontent.com/hts0000/images/main/202211081713311.png)
![](https://raw.githubusercontent.com/hts0000/images/main/202211081714284.png)

## 维护数据库
### 备份数据库
```sql
# 现在mysql命令行中执行，保证所有数据都写盘了
FLUSH TABLES;
# 再用备份工具备份
mysqldump
```

