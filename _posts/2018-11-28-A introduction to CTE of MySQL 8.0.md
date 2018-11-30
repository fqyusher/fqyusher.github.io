---
layout: post
title:  "MySQL 8.0 新特性--CTE介绍"
date:   2018-11-28 14:55:46 +0800
category: translation
tags: [mysql]
excerpt: "介绍MySQL 8.0新特性--公用表表达式(CTE)的用法"
---
>原文地址：[http://www.mysqltutorial.org/mysql-cte/](http://www.mysqltutorial.org/mysql-cte/)

# 总结

在这篇文章中，你将学会如何使用MySQL的公用表表达式(CTE:common table expression)来构建更易阅读的复杂语句。

# 什么是公用表表结构(CTE)

CTE是一个以临时结果集命名的概念，它仅存在于单个SQL语句的执行范围内， 如：SELECT、INSERT、UPDATE或者DELETE。

与派生表类似，一个CTE不作为存储对象，仅在执行期间持续存在。不同的是，CTE可以是自引用(递归CTE)，也可在同一个查询中被多次引用。除此之外，CTE提供了更好的可读性和性能。

# CTE语法
一个CTE的结构包括名字，可选列名列表以及定义CTE的查询。定义完CTE后，您可以像声明SELECT、INSERT、UPDATE、DELETE或CREATE视图一样来使用它。

如下举例说明了CTE的基本语法：

```sql
WITH cte_name (column_list) AS (
    query
) 
SELECT * FROM cte_name;
```

清注意，在上面的query语句中的列数必须与column_list的数量一致。如果你忽略了column_list，CTE将使用CTE定义的query语句中的列列表。

# 简单的CTE例子

下面的例子说明了如何使用CTE从示例数据库中的customer表中查询所需数据。这个例子仅作为示范操作，目的是为了让您更好的理解CTE概念。

```sql
WITH customers_in_usa AS (
    SELECT 
        customerName, state
    FROM
        customers
    WHERE
        country = 'USA'
)
SELECT 
    customerName
FROM
    customers_in_usa
WHERE
    state = 'CA'
ORDER BY customerName;
```
查询结果：

![](/images/MySQL-CTE-Example-1.png)

在这个例子中，CTE的名称是customers_in_usa，查询语句返回customerName和state两个列。因此，该CTE返回位于美国的所有顾客。
定义完customers_in_usa后，我们通过引用SELECT声明语句来获取位于California的所有顾客。

请看另一个例子：

```sql
WITH topsales2003 AS (
    SELECT 
        salesRepEmployeeNumber employeeNumber,
        SUM(quantityOrdered * priceEach) sales
    FROM
        orders
            INNER JOIN
        orderdetails USING (orderNumber)
            INNER JOIN
        customers USING (customerNumber)
    WHERE
        YEAR(shippedDate) = 2003
            AND status = 'Shipped'
    GROUP BY salesRepEmployeeNumber
    ORDER BY sales DESC
    LIMIT 5
)
SELECT 
    employeeNumber, firstName, lastName, sales
FROM
    employees
        JOIN
    topsales2003 USING (employeeNumber);
```

查询结果：

![](/images/MySQL-CTE-Example-2.png)

在这个例子中，CTE返回在2003年的前五名推销员。之后，我们引用该CTE(topsales2003)来获取关于这些推销员的额外信心，包括名字和姓氏。

# 一个更复杂的CTE例子

请看如下例子：

```sql
WITH salesrep AS (
    SELECT 
        employeeNumber,
        CONCAT(firstName, ' ', lastName) AS salesrepName
    FROM
        employees
    WHERE
        jobTitle = 'Sales Rep'
),
customer_salesrep AS (
    SELECT 
        customerName, salesrepName
    FROM
        customers
            INNER JOIN
        salesrep ON employeeNumber = salesrepEmployeeNumber
)
SELECT 
    *
FROM
    customer_salesrep
ORDER BY customerName;
```
查询结果：

![](/images/MySQL-CTE-Example-3.png)

在这个例子中，在同一个查询一句中我们有两个CTE。第一个CTE(salesrep)用于获取职位是推销员的雇员。第二个CTE(customer_salesrep)通过引用第一个CTE(salesrep)以及INNER JOIN分句来获取推销员以及他所负责的顾客。

在有了第二个CTE之后，我们可以使用简单的包含ORDER BY的SELECT语句来获取数据。

# WITH分句的用法

有些上下文可以使用WITH分句来创建CTE：

1. 可以用在SELECT、UPDATE以及DELETE语句的开头：

```sql
WITH ... SELECT ...
WITH ... UPDATE ...
WITH ... DELETE ...
```

2. 可以用在子查询或者派生表子查询的开头：

```sql
SELECT ... WHERE id IN (WITH ... SELECT ...);
 
SELECT * FROM (WITH ... SELECT ...) AS derived_table;
```

3. 可以用在包含SELECT子句的声明语句的SELECT之前

```sql
CREATE TABLE ... WITH ... SELECT ...
CREATE VIEW ... WITH ... SELECT ...
INSERT ... WITH ... SELECT ...
REPLACE ... WITH ... SELECT ...
DECLARE CURSOR ... WITH ... SELECT ...
EXPLAIN ... WITH ... SELECT ...
```

在这篇文章中，你已经学会了如何使用MySQL的公用表表达式。

# 相关文章

* [Managing Hierarchical Data in MySQL Using the Adjacency List Model](http://www.mysqltutorial.org/mysql-adjacency-list-tree/)
* [A Definitive Guide To MySQL Recursive CTE](http://www.mysqltutorial.org/mysql-recursive-cte/)