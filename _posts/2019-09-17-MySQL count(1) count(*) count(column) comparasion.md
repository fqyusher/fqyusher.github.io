---
layout: post
title:  "MySQL-count(1), count(*), count(column)对比"
date:   2019-09-17 15:22:46 +0800
category: [技能, 原创]
tags: [mysql]
excerpt: "MySQL-count(1), count(*), count(column)对比"
---

# 概述

本文将对MySQL的`count(1)`, `count(*)`, `count(column)`进行对比分析。

## 作用与区别

> 作用

-   **count(1):** 统计表中所有记录数，包括字段为`null`的记录。
-   **count(*):** 与`count(1)`作用相同，统计表中所有记录，包括字段为`null`的记录。
-   **count(column):** 统计该字段在表中出现的次数，忽略该字段为`null`的记录，但不会忽略0或空字符串的记录。

> 区别

-   `count(column)`中的column若不为primary key时，会忽略column为`null`的记录。
-   效率上`count(*)`与`count(1)`相差不大，`count(pk)`次之，`count(non-index)`最慢。

## 验证-数据表无null值或空值

> 表结构

![](/images/table-desc.jpg)

> count结果

![](/images/table-count-result.jpg)

> count对比图

![](/images/table-count-comparasion.jpg)

## 验证-数据表存在null值或者空值

> 更新部分字段为null或空值

![](/images/table-update.jpg)

> count结果

![](/images/table-count-result-2.jpg)

> count对比图

![](/images/table-count-comparasion-2.jpg)


## 验证-name字段具有索引

> 添加`name`字段索引

![](/images/table-add-index.jpg)

> count对比图

![](/images/table-count-comparasion-3.jpg)
