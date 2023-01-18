# MySQL Handling of GROUP BY

> 摘要信息: 
>
> 对比`MySQL`和标准`SQL`语句之间在针对字段别名等情况, `group by`的允许范围差异. 
>
> select语句后返回的字段内容, 在`only_full_group_by`模式下, 假如这个字段没有纳入`group by`聚合的字段中, 将会出现错误.类似的, 假如使用聚合函数, 如max(), 同时返回没有其他的字段也会引发类似的问题.
>
> - 关闭这种模式(不建议).
> - 在非聚合的要返回的字段, 使用`any_value()`, 返回随意值.
> - 假如非聚合的字段是主键或者是非`null`的唯一索引, 则没有问题.

SQL-92 and earlier does not permit queries for which the select list, `HAVING` condition, or `ORDER BY` list refer to nonaggregated columns that are not named in the `GROUP BY` clause. For example, this query is illegal in standard SQL-92 because the nonaggregated `name` column in the select list does not appear in the `GROUP BY`:

`SQL-92`以及更早之前的标准不允许`select`、`having`条件语句或`order by`列表引用不在`group by`子句中命名的非聚合列的查询.

这种查询语句是非法的在`SQL-92`标准中, 因为未聚合的列`name`没有出现在`group by`语句中.

```sql
SELECT o.custid, c.name, MAX(o.payment)
  FROM orders AS o, customers AS c
  WHERE o.custid = c.custid
  GROUP BY o.custid;
```

```bash
# 见这个例子
mysql> select * from test;
+------+------+------------+
| id   | col  | c_year     |
+------+------+------------+
|    3 | c    | 1991-01-02 |
|    4 | d    | 2004-01-01 |
|    1 | a    | 2011-01-02 |
|    2 | b    | 2022-11-04 |
+------+------+------------+
4 rows in set (0.00 sec)

mysql> select c_year, sum(id) from test group by col;
ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'test_db.test.c_year' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

For the query to be legal in SQL-92, the `name` column must be omitted from the select list or named in the `GROUP BY` clause.

对于上述的查询语句, 要满足SQL-2标准的要求, `name`列必须从select语句中移除掉或者加入到`group by`语句中去.

SQL:1999 and later permits such nonaggregates per optional feature T301 if they are functionally dependent on `GROUP BY` columns: If such a relationship exists between `name` and `custid`, the query is legal. This would be the case, for example, were `custid` a primary key of `customers`.

MySQL implements detection of functional dependence. If the [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by) SQL mode is enabled (which it is by default), MySQL rejects queries for which the select list, `HAVING` condition, or `ORDER BY` list refer to nonaggregated columns that are neither named in the `GROUP BY` clause nor are functionally dependent on them.

MySQL会检测函数的依赖. 当 `only_full_group`模式启用(这是默认开启的), MySQL也会拒接执行上述的例子.

MySQL also permits a nonaggregate column not named in a `GROUP BY` clause when SQL [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by) mode is enabled, provided that this column is limited to a single value, as shown in the following example:

 当`only_full_group`处于启用时, MySQL允许非聚合的字段不出现在`group by`语句中, 这只对单个值字段, 如下面的例子.

```sql
mysql> CREATE TABLE mytable (
    ->    id INT UNSIGNED NOT NULL PRIMARY KEY,
    ->    a VARCHAR(10),
    ->    b INT
    -> );

mysql> INSERT INTO mytable
    -> VALUES (1, 'abc', 1000),
    ->        (2, 'abc', 2000),
    ->        (3, 'def', 4000);

mysql> SET SESSION sql_mode = sys.list_add(@@session.sql_mode, 'ONLY_FULL_GROUP_BY');

mysql> SELECT a, SUM(b) FROM mytable WHERE a = 'abc';
+------+--------+
| a    | SUM(b) |
+------+--------+
| abc  |   3000 |
+------+--------+
```

It is also possible to have more than one nonaggregate column in the [`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html) list when employing [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by). In this case, every such column must be limited to a single value in the `WHERE` clause, and all such limiting conditions must be joined by logical `AND`, as shown here:

MySQL同样支持在超过非聚合的多个字段在`select`语句中,在`only_full_group`模式下. 在这个例子中, 同样限制单个值, 在where语句中使用 and操作符进行连接. 如下面的例子.

```sql
mysql> DROP TABLE IF EXISTS mytable;

mysql> CREATE TABLE mytable (
    ->    id INT UNSIGNED NOT NULL PRIMARY KEY,
    ->    a VARCHAR(10),
    ->    b VARCHAR(10),
    ->    c INT
    -> );

mysql> INSERT INTO mytable
    -> VALUES (1, 'abc', 'qrs', 1000),
    ->        (2, 'abc', 'tuv', 2000),
    ->        (3, 'def', 'qrs', 4000),
    ->        (4, 'def', 'tuv', 8000),
    ->        (5, 'abc', 'qrs', 16000),
    ->        (6, 'def', 'tuv', 32000);

mysql> SELECT @@session.sql_mode;
+---------------------------------------------------------------+
| @@session.sql_mode                                            |
+---------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION |
+---------------------------------------------------------------+

mysql> SELECT a, b, SUM(c) FROM mytable
    ->     WHERE a = 'abc' AND b = 'qrs';
+------+------+--------+
| a    | b    | SUM(c) |
+------+------+--------+
| abc  | qrs  |  17000 |
+------+------+--------+
```

If [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by) is disabled, a MySQL extension to the standard SQL use of `GROUP BY` permits the select list, `HAVING` condition, or `ORDER BY` list to refer to nonaggregated columns even if the columns are not functionally dependent on `GROUP BY` columns. This causes MySQL to accept the preceding query. In this case, the server is free to choose any value from each group, so unless they are the same, the values chosen are nondeterministic, which is probably not what you want. Furthermore, the selection of values from each group cannot be influenced by adding an `ORDER BY` clause. Result set sorting occurs after values have been chosen, and `ORDER BY` does not affect which value within each group the server chooses. Disabling [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by) is useful primarily when you know that, due to some property of the data, all values in each nonaggregated column not named in the `GROUP BY` are the same for each group.

假如`only_full_group`被禁用, MySQL将延展标准的SQL语句, 允许上述的非法操作. 但是需要注意, 返回的结果未必是你所期待的.

You can achieve the same effect without disabling [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by) by using [`ANY_VALUE()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_any-value) to refer to the nonaggregated column.

可以使用`any_value`来实现不需要关闭`only_full_group_by`模式.

The following discussion demonstrates functional dependence, the error message MySQL produces when functional dependence is absent, and ways of causing MySQL to accept a query in the absence of functional dependence.

下面的讨论, 将演示函数依赖`MySQL`在不存在函数依赖时产生的错误消息，以及导致MySQL在不存在函数依赖时接受查询的方法.

This query might be invalid with [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by) enabled because the nonaggregated `address` column in the select list is not named in the `GROUP BY` clause:

在`only_full_group`模式下, 这条语句是无效的, 因为非聚合的字段, `address`出现在select语句中, 而这个字段并未出现在`group by`中.

```sql
SELECT name, address, MAX(age) FROM t GROUP BY name;
```

The query is valid if `name` is a primary key of `t` or is a unique `NOT NULL` column. In such cases, MySQL recognizes that the selected column is functionally dependent on a grouping column. For example, if `name` is a primary key, its value determines the value of `address` because each group has only one value of the primary key and thus only one row. As a result, there is no randomness in the choice of `address` value in a group and no need to reject the query.

假如`name`字段是主键或者是非空的唯一索引, 那么这条语句是有效的.

The query is invalid if `name` is not a primary key of `t` or a unique `NOT NULL` column. In this case, no functional dependency can be inferred and an error occurs:

反之, 则会引发以下的错误.

```sql
mysql> SELECT name, address, MAX(age) FROM t GROUP BY name;
ERROR 1055 (42000): Expression #2 of SELECT list is not in GROUP
BY clause and contains nonaggregated column 'mydb.t.address' which
is not functionally dependent on columns in GROUP BY clause; this
is incompatible with sql_mode=only_full_group_by
```

If you know that, *for a given data set,* each `name` value in fact uniquely determines the `address` value, `address` is effectively functionally dependent on `name`. To tell MySQL to accept the query, you can use the [`ANY_VALUE()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_any-value) function:

如你所知道的, 在给定的数据集中, name字段应当决定address`字段的唯一值, `address`字段应当依赖于`name`字段. 所以可以告诉MySQL可以返回任意值, 使用any_value函数.

```sql
SELECT name, ANY_VALUE(address), MAX(age) FROM t GROUP BY name;
```

Alternatively, disable [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by).

否则, 就只能关闭`only_full_group_by`模式.

The preceding example is quite simple, however. In particular, it is unlikely you would group on a single primary key column because every group would contain only one row. For additional examples demonstrating functional dependence in more complex queries, see [Section 12.20.4, “Detection of Functional Dependence”](https://dev.mysql.com/doc/refman/8.0/en/group-by-functional-dependence.html).

然而, 上述的例子都是相对简单的. 

If a query has aggregate functions and no `GROUP BY` clause, it cannot have nonaggregated columns in the select list, `HAVING` condition, or `ORDER BY` list with [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by) enabled:

```sql
mysql> SELECT name, MAX(age) FROM t;
ERROR 1140 (42000): In aggregated query without GROUP BY, expression
#1 of SELECT list contains nonaggregated column 'mydb.t.name'; this
is incompatible with sql_mode=only_full_group_by
```

Without `GROUP BY`, there is a single group and it is nondeterministic which `name` value to choose for the group. Here, too, [`ANY_VALUE()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_any-value) can be used, if it is immaterial which `name` value MySQL chooses:

```sql
SELECT ANY_VALUE(name), MAX(age) FROM t;
```

`ONLY_FULL_GROUP_BY` also affects handling of queries that use `DISTINCT` and `ORDER BY`. Consider the case of a table `t` with three columns `c1`, `c2`, and `c3` that contains these rows:

注意这种模式对于`distinct`的影响

```none
c1 c2 c3
1  2  A
3  4  B
1  2  C
```

Suppose that we execute the following query, expecting the results to be ordered by `c3`:

假设, 执行下面这条语句

```sql
SELECT DISTINCT c1, c2 FROM t ORDER BY c3;
```

To order the result, duplicates must be eliminated first. But to do so, should we keep the first row or the third? This arbitrary choice influences the retained value of `c3`, which in turn influences ordering and makes it arbitrary as well. To prevent this problem, a query that has `DISTINCT` and `ORDER BY` is rejected as invalid if any `ORDER BY` expression does not satisfy at least one of these conditions:

- The expression is equal to one in the select list
- All columns referenced by the expression and belonging to the query's selected tables are elements of the select list

Another MySQL extension to standard SQL permits references in the `HAVING` clause to aliased expressions in the select list. For example, the following query returns `name` values that occur only once in table `orders`:

```sql
SELECT name, COUNT(name) FROM orders
  GROUP BY name
  HAVING COUNT(name) = 1;
```

The MySQL extension permits the use of an alias in the `HAVING` clause for the aggregated column:

MySQL支持在having后使用字段别名

```sql
SELECT name, COUNT(name) AS c FROM orders
  GROUP BY name
  HAVING c = 1;
```

在有having, 筛选条件下使用, 在MySQL是允许的.

Standard SQL permits only column expressions in `GROUP BY` clauses, so a statement such as this is invalid because `FLOOR(value/100)` is a noncolumn expression:

在标准的SQL中只允许group by聚合字段, 类似于下面的语句也是无效的.(虽然看起来临时字段也出现在group by后面)

```sql
SELECT id, FLOOR(value/100)
  FROM tbl_name
  GROUP BY id, FLOOR(value/100);
```

MySQL extends standard SQL to permit noncolumn expressions in `GROUP BY` clauses and considers the preceding statement valid.

Standard SQL also does not permit aliases in `GROUP BY` clauses. MySQL extends standard SQL to permit aliases, so another way to write the query is as follows:

同样标准SQL不允许别名的形式, 尽管别名也出现在group by后.

```sql
SELECT id, FLOOR(value/100) AS val
  FROM tbl_name
  GROUP BY id, val;
```

The alias `val` is considered a column expression in the `GROUP BY` clause.

别名`val`被视作字段, 在`group by`中.

In the presence of a noncolumn expression in the `GROUP BY` clause, MySQL recognizes equality between that expression and expressions in the select list. This means that with [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by) SQL mode enabled, the query containing `GROUP BY id, FLOOR(value/100)` is valid because that same [`FLOOR()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_floor) expression occurs in the select list. However, MySQL does not try to recognize functional dependence on `GROUP BY` noncolumn expressions, so the following query is invalid with [`ONLY_FULL_GROUP_BY`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by) enabled, even though the third selected expression is a simple formula of the `id` column and the [`FLOOR()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_floor) expression in the `GROUP BY` clause:

这个语句在`only_full_group_by`模式下的MySQL是合法的.(有别于上面的标准SQL)

```sql
SELECT id, FLOOR(value/100), id+FLOOR(value/100)
  FROM tbl_name
  GROUP BY id, FLOOR(value/100);
```

A workaround is to use a derived table:

衍生表中使用:

```sql
 SELECT id, F, id+F
  FROM
    (SELECT id, FLOOR(value/100) AS F
     FROM tbl_name
     GROUP BY id, FLOOR(value/100)) AS dt;
```