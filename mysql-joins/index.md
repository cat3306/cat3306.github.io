# mysql joins




# LEFT JOIN

`LEFT JOIN` 可将两张或两张以上的表以某种条件连接起来，最左边的表作为主表，故叫`LEFT JOIN`

基本语法

```sql
SELECT 
    select_list
FROM
    t1
LEFT JOIN t2 ON 
    join_condition;
```

在上面的语法中，`t1`是左表(主表)，t2是右表

`LEFT JOIN` 从左表开始(`t1`)筛选数据，并且`t1`的每一条与`t2`的每一条基于`join_condition`做判断，如此一来会有两种情况

- 如果`join_condition`的值为`true`,则`LEFT JOIN`两个表中行合并成一个新的数据行
- 如果`join_condition`的值为`false`，即`t1`的数据与`t2`的任何一条都不匹配，依然生成新的一条数据，但是`t2`的所有列都是`NULL`

也就是说`LEFT JOIN`返回的是`t1`所有的数据条数，而不管是否与`t2`的数据匹配，能匹配则带上`t2`的数据，不能匹配用`NULL`占位

{{% figure  src="/img/mysql/24.png" alt="img" alt="img" %}}

## 例子

[Download MySQL Sample Database](https://www.mysqltutorial.org/wp-content/uploads/2018/03/mysqlsampledatabase.zip)

### 两张表

{{% figure  src="/img/mysql/26.png" alt="img" alt="img" %}}

这是顾客表和订单表

```sql
SELECT 
    c.`customerNumber`, 
    c.`customerName`, 
    o.`orderNumber`, 
    o.`status`
FROM
    customers as c
LEFT JOIN orders as o ON 
    c.customerNumber = o.customerNumber;
```

{{% figure  src="/img/mysql/25.png" alt="img" alt="img" %}}

可以看见有些数据的右表字段是`NULL`

利用这个特性，很容知道哪些顾客没有订单

```sql
SELECT 
    c.`customerNumber`, 
    c.`customerName`, 
    o.`orderNumber`, 
    o.`status`
FROM
    customers as c
LEFT JOIN orders as o ON 
    c.customerNumber = o.customerNumber
    WHERE o.`orderNumber` IS NULL
```

{{% figure  src="/img/mysql/27.png" alt="img" alt="img" %}}

### 三张表

如果是三张表，情况如下:(far fa-hand-point-down fa-fw):

{{% figure  src="/img/mysql/28.png" alt="img" alt="img" %}}

```sql
SELECT 
    e.lastName, 
    e.firstName, 
    c.customerName, 
    p.checkNumber, 
    p.amount
FROM
    employees as e
LEFT JOIN customers as c ON 
    e.employeeNumber = c.salesRepEmployeeNumber
LEFT JOIN payments as p ON 
    p.customerNumber = c.customerNumber
ORDER BY 
    c.customerName, 
    p.checkNumber;
```

{{% figure  src="/img/mysql/29.png" alt="img" alt="img" %}}

- 第一次`LEFT JOIN`是`employees`和`customers`关联，当关联的条件为`false`时，`customers`部分字段用`NULL`表示
- 第二次`LEFT JOIN`是`customers`和`payments`关联，当关联的条件为`false`时，`payments`部分字段用`NULL`表示

可以看到第一次关联不到数据项，第二次也关联不上，即如果`customerName`为`NULL`则`payments`部分字段必为`NULL`

### 过滤条件应该放哪儿

将过滤条件放在`WHERE`中

```sql
SELECT 
    o.orderNumber, 
    customerNumber, 
    productCode
FROM
    orders as o
LEFT JOIN orderDetails as od
   ON o.orderNumber=od.orderNumber
WHERE
    o.orderNumber = 10123;
```

{{% figure  src="/img/mysql/30.png" alt="img" alt="img" %}}



将过滤条件放在`ON`中

```sql
SELECT
	o.orderNumber,
	customerNumber,
	productCode 
FROM
	orders AS o
	LEFT JOIN orderDetails AS od ON o.orderNumber = od.orderNumber 
	AND o.orderNumber = 10123;
```

{{% figure  src="/img/mysql/31.png" alt="img" alt="img" %}}

看出差别来了吗

过滤条件放在连接语句的`ON`里面，是作为一种连接条件，只会关联`o.orderNumber = od.orderNumber ` 并且`o.orderNumber = 10123`,但是左表依然是全部查询，只是关联条件比原来(`o.orderNumber = od.orderNumber`)苛刻了

而放在`WHERE`里面是将连接表作为整体来看，只展示`o.orderNumber = 10123`的数据

# RIGHT JOIN

`RIGHT JOIN` 和 `LEFT JOIN` 非常类似，唯一的区别是哪个表作为主表

```sql
SELECT 
    select_list
FROM t1
RIGHT JOIN t2 ON 
    join_condition;
```

在上面:(far fa-hand-point-up fa-fw):的`sql`中，`t2`作为主表



```sql
SELECT 
    employeeNumber, 
    customerNumber
FROM
    customers
RIGHT JOIN employees 
    ON salesRepEmployeeNumber = employeeNumber
ORDER BY 
	employeeNumber;
```

{{% figure  src="/img/mysql/32.png" alt="img" alt="img" %}}

# INNER JOIN

`INNER JOIN` 取几个表的交集

```sql
SELECT
    select_list
FROM t1
INNER JOIN t2 ON join_condition1
INNER JOIN t3 ON join_condition2
...;
```



{{% figure  src="/img/mysql/33.png" alt="img" alt="img" %}}

### 例子

```sql
SELECT 
    c.`customerNumber`, 
    c.`customerName`, 
    o.`orderNumber`, 
    o.`status`
FROM
    customers as c
INNER JOIN orders as o ON 
    c.customerNumber = o.customerNumber
    WHERE `status` IS NULL;
```

{{% figure  src="/img/mysql/34.png" alt="img" alt="img" %}}

可以发现，`WHERE status IS NULL ` 的结果集为空，这和`LEFT JOIN`不一样，这是因为INNER JOIN 只有`join_condition`为`true`时才会生成新的数据行

可以和`LEFT JOIN` (`RIGHT JOIN`)搭配起来使用

`employees`表(主表)和`customers`表内联，和`payments`左联

```sql
SELECT 
    e.lastName, 
    e.firstName, 
    c.customerName, 
    p.checkNumber, 
    p.amount
FROM
    employees as e
INNER JOIN customers as c ON 
    e.employeeNumber = c.salesRepEmployeeNumber
LEFT JOIN payments as p ON 
    p.customerNumber = c.customerNumber
ORDER BY 
    c.customerName, 
    p.checkNumber;
```

{{% figure  src="/img/mysql/35.png" alt="img" alt="img" %}}



`employees`表(主表)和`customers`表左联，和`payments`内联

```sql
SELECT 
    e.lastName, 
    e.firstName, 
    c.customerName, 
    p.checkNumber, 
    p.amount
FROM
    employees as e
LEFT JOIN customers as c ON 
    e.employeeNumber = c.salesRepEmployeeNumber
INNER JOIN payments as p ON 
    p.customerNumber = c.customerNumber
ORDER BY 
    c.customerName, 
    p.checkNumber;
```


