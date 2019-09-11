---
layout: post
title:  "MySQL Join的使用介绍"
date:   2019-09-11 16:22:46 +0800
category: skill
tags: [mysql]
excerpt: "MySQL Join使用方式介绍"
---


# 概述

本文将介绍MySQL的各种Join的使用方式。

## 表结构与数据

>   table1

| id   | name |
| ---- | ---- |
| 1    | Rose |
| 2    | Kobe |
| 8    | John |

>   table2

| id   | name   |
| ---- | ------ |
| 1    | Wade   |
| 2    | Kobe   |
| 3    | Durant |
| 4    | Curry  |

## Join使用类型

### 内联接(INNER JOIN)

>   图示

![](/images/inner-join.jpg)

>   SQL语句

```sql
# inner join
SELECT a.*, b.* FROM table1 a
INNER JOIN table2 b ON a.id = b.id;
```

>   结果

| id   | name | id(1) | name(1) |
| ---- | ---- | ----- | ------- |
| 1    | Rose | 1     | Wade    |
| 2    | Kobe | 2     | Kobe    |

### 左连接(LEFT [OUTER] JOIN)

>   图示

![](/images/left-join.jpg)

>   Sql语句

```sql
# left join
SELECT a.*, b.* FROM table1 a
LEFT JOIN table2 b ON a.id = b.id;
```

>   结果

| id   | name | id(1) | name(1) |
| ---- | ---- | ----- | ------- |
| 1    | Rose | 1     | Wade    |
| 2    | Kobe | 2     | Kobe    |
| 8    | John | Null  | Null    |

### 右连接(RIGHT [OUTER] JOIN)

>   图示

![](/images/right-join.jpg)

>   SQL语句

```sql
# right-join
SELECT a.*, b.* FROM table1 a
RIGHT JOIN table2 b ON a.id = b.id;
```

>   结果

| id   | name | id(1) | name(1) |
| ---- | ---- | ----- | ------- |
| 1    | Rose | 1     | Wade    |
| 2    | Kobe | 2     | Kobe    |
| Null | Null | 3     | Durant  |
| Null | Null | 4     | Curry   |

### 差集(Intersect)

>   图示

![](/images/intersect-left.jpg)

>   SQL语句

```sql
# intersection: table1 - table2(属于table1但不属于table2)
SELECT a.*, b.* FROM table1 a
LEFT JOIN table2 b ON a.id = b.id
WHERE b.id IS NULL
```

>   结果

| id   | name | id(1) | name(1) |
| ---- | ---- | ----- | ------- |
| 8    | John | Null  | Null    |

>   图示

![](/images/intersect-right.jpg)

>   SQL语句

```sql
# intersection: table2 - table1(属于table2但不属于table1)
SELECT a.*, b.* FROM table1 a
RIGHT JOIN table2 b ON a.id = b.id
WHERE a.id IS NULL;
```

>   结果

| id   | name | id(1) | name(1) |
| ---- | ---- | ----- | ------- |
| Null | Null | 3     | Durant  |
| Null | Null | 4     | Curry   |

### 并集(UNION)

>   图示

![](/images/union-join.jpg)

>   SQL语句

```sql
# union join
SELECT a.*, b.* FROM table1 a
LEFT JOIN table2 b ON a.id = b.id
UNION
SELECT a.*, b.* FROM table1 a
RIGHT JOIN table2 b ON a.id = b.id;
```

>   结果

| id   | name | id(1) | name(1) |
| ---- | ---- | ----- | ------- |
| 1    | Rose | 1     | Wade    |
| 2    | Kobe | 2     | Kobe    |
| 8    | John | Null  | Null    |
| Null | Null | 3     | Durant  |
| Null | Null | 4     | Curry   |

### 对称差(Semmetric Difference)

>   图示

![](/images/symmetric-difference.jpg)

>   SQL语句

```sql
# symmetric difference
SELECT a.*, b.* FROM table1 a
LEFT JOIN table2 b ON a.id = b.id
WHERE b.id IS NULL
UNION
SELECT a.*, b.* FROM table1 a
RIGHT JOIN table2 b ON a.id = b.id
WHERE a.id IS NULL;
```

>   结果

| id   | name | id(1) | name(1) |
| ---- | ---- | ----- | ------- |
| 8    | John | Null  | Null    |
| Null | Null | 3     | Durant  |
| Null | Null | 4     | Curry   |

