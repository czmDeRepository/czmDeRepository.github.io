---
title: sql中distinct简单使用
date: 2020-09-22 11:32:13
tags: 
  - sql
categories: 数据库
---



### **简介：distinct常用于sql语句中查出重复数据时需要去重的时候**

<!--distinct不会过滤掉null值，返回结果包含null值-->

示例：table

| column1 | column2 |
| ------- | ------- |
| 1       | a       |
| 1       | b       |
| 2       | c       |
| 2       | c       |
| 3       | d       |
| null    | e       |
| null    | f       |
|         | g       |

### **只有一字段**：

​		**直接在字段前加distinct**

​		select distinct column1 from table

结果：

| column1 |
| ------- |
| 1       |
| 2       |
| 3       |
| null    |
|         |

### **需要查出多个字段**：

​	distinct 需要放在第一个字段前面，否者报错

select distinct colum1 , colum2 from table;

结果：多个字段其实是多个字段查出后拼接再去重

| column1 | column2 |
| ------- | ------- |
| 1       | a       |
| 1       | b       |
| 2       | c       |
| 3       | d       |
| null    | e       |
| null    | f       |
|         | g       |



### **只想一个字段唯一**

**1.使用group by**

​	select column1, column2 from table group by column1

**2.使用GROUP_CONCAT函数**

​	**GROUP_CONCAT在连接查询的时候，能让查出的这字段的多个数，按字符拼接的方式存放在一起。**

select GROUP_CONCAT(distinct column1) as column1, column2 from table group by column1

| column1 | column2 |
| ------- | ------- |
| 1       | a       |
| 2       | c       |
| 3       | d       |
| null    | e       |
|         | g       |

### END