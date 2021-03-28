---
title: MySQL-Explain 语法备忘
date: 2020-05-20 08:48:14
tags: 
  - MySQL Explain
categories:
  - mysql
---



### explain 语法

```sql
explain select * from mysql.user;
```

### explain 输出

| id   | select\_type | table | partitions | type | possible\_keys | key  | key\_len | ref  | rows | filtered | Extra |
| :--- | :----------- | :---- | :--------- | :--- | :------------- | :--- | :------- | :--- | :--: | :------- | :---- |
| 1    | SIMPLE       | NULL  | NULL       | NULL | NULL           | NULL | NULL     | NULL | NULL | NULL     |       |

#### 每列简单含义

| Column                                                       | JSON Name       | Meaning                                                      |
| ------------------------------------------------------------ | --------------- | :----------------------------------------------------------- |
| [`id`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_id) | `select_id`     | 执行编号                                                     |
| [`select_type`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_select_type) | None            | 每个执行的类型(SIMPLE\|PRIMARY\|UNION\|SUBQUERY\|DERIVED)    |
| [`table`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_table) | `table_name`    | 每个执行的表名(实际表名或引用其他执行)                       |
| [`partitions`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_partitions) | `partitions`    | The matching partitions                                      |
| [`type`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_type) | `access_type`   | The join type                                                |
| [`possible_keys`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_possible_keys) | `possible_keys` | 预测会使用的索引                                             |
| [`key`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_key) | `key`           | The index actually chosen                                    |
| [`key_len`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_key_len) | `key_length`    | The length of the chosen key                                 |
| [`ref`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_ref) | `ref`           | The columns compared to the index                            |
| [`rows`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_rows) | `rows`          | 表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数 |
| [`filtered`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_filtered) | `filtered`      | 按表条件过滤的行百分比                                       |
| [`Extra`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_extra) | None            | 包含MySQL解决查询的详细信息                                  |

#### 含义详解

##### select_type 

```html
(1) SIMPLE(简单SELECT,不使用UNION或子查询等)
(2) PRIMARY(查询中若包含任何复杂的子部分,最外层的select被标记为PRIMARY)
(3) UNION(UNION中的第二个或后面的SELECT语句)
(4) DEPENDENT UNION(UNION中的第二个或后面的SELECT语句，取决于外面的查询)
(5) UNION RESULT(UNION的结果)
(6) SUBQUERY(子查询中的第一个SELECT)
(7) DEPENDENT SUBQUERY(子查询中的第一个SELECT，取决于外面的查询)
(8) DERIVED(派生表的SELECT, FROM子句的子查询)
(9) UNCACHEABLE SUBQUERY(一个子查询的结果不能被缓存，必须重新评估外链接的第一行)
```

##### type

```html
(1) ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行
(2) index: Full Index Scan，index与ALL区别为index类型只遍历索引树
(3) range:只检索给定范围的行，使用一个索引来选择行
(4) ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值
(5) eq_ref: 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件
(6) const、system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量,system是const类型的特例，当查询的表只有一行的情况下，即为system
(7) NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。
```

参考文档: https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain-join-types

##### extra

```bash
(1) Using where:列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示mysql服务器将在存储引擎检索行后再进行过滤
(2) Using temporary：表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询
(3) Using filesort：MySQL中无法利用索引完成的排序操作称为“文件排序”
(4) Using join buffer：改值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进能。（如给关联表即行记录的table的关联列加索引）
(5) Impossible where：这个值强调了where语句会导致没有符合条件的行。
(6) Select tables optimized away：这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行
(7) Using Index: 仅使用索引树中的信息从表中检索列信息，而不必进行其他查找以读取实际行。当查询仅使用属于单个索引的列时，可以使用此策略。
```

参考文档: https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain-extra-information

#### 总结

> 一般先针对 type, 将 type 调优到 ref, range, index 级别
>
> 然后查看 extra 信息, 尽量消除 using filesort, using join buffer, using temporary, using where
>
> 针对复杂的查询条件, 建议添加复合索引且确保复合索引有效使用(不跨列,按顺序,最左原则)

#### 注意事项

```bash
1.Using Join Buffer: 对关联列加索引是没错, 但记住是关联表加, 而不是主表.
2.使用Explain时, SQL语句中的条件最好不要太真实, 可能会影响分析(如一张表全部都是sex=男, sex加索引, 然后使用explain分析时带上where sex='男', 分析结果的type会是ALL, 但如果sex='', 则type为ref)
```

