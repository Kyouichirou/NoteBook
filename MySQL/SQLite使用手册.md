## SQLite简单使用手册

## 数据类型

虽然`SQLite`和`MySQL`一样在名字上有`SQL`, 但是`SQLite`在很多方面和`MySQL`有所差异

`SQLite`原生支持5中数据类型: `NULL`, `INTEGER`, `REAL`, `TEXT`, `BLOB`. 在`SQLite`中, **所有数据最终都转化为该5中类型进行存储**.

| 存储类  | 描述                                                         |
| :------ | :----------------------------------------------------------- |
| NULL    | 值是一个 NULL 值。                                           |
| INTEGER | 值是一个带符号的整数，根据值的大小存储在 1、2、3、4、6 或 8 字节中。 |
| REAL    | 值是一个浮点值，存储为 8 字节的 IEEE 浮点数字。              |
| TEXT    | 值是一个文本字符串，使用数据库编码（UTF-8、UTF-16BE 或 UTF-16LE）存储。 |
| BLOB    | 值是一个 blob 数据，完全根据它的输入存储。                   |

## SQLite 亲和(Affinity)类型

SQLite支持列的亲和类型概念。任何列仍然可以存储任何类型的数据，当数据插入时，该字段的数据将会优先采用亲缘类型作为该值的存储方式。SQLite目前的版本支持以下五种亲缘类型：

| 亲和类型 | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| TEXT     | 数值型数据在被插入之前，需要先被转换为文本格式，之后再插入到目标字段中。 |
| NUMERIC  | 当文本数据被插入到亲缘性为NUMERIC的字段中时，如果转换操作不会导致数据信息丢失以及完全可逆，那么SQLite就会将该文本数据转换为INTEGER或REAL类型的数据，如果转换失败，SQLite仍会以TEXT方式存储该数据。对于NULL或BLOB类型的新数据，SQLite将不做任何转换，直接以NULL或BLOB的方式存储该数据。需要额外说明的是，对于浮点格式的常量文本，如"30000.0"，如果该值可以转换为INTEGER同时又不会丢失数值信息，那么SQLite就会将其转换为INTEGER的存储方式。 |
| INTEGER  | 对于亲缘类型为INTEGER的字段，其规则等同于NUMERIC，唯一差别是在执行CAST表达式时。 |
| REAL     | 其规则基本等同于NUMERIC，唯一的差别是不会将"30000.0"这样的文本数据转换为INTEGER存储方式。 |
| NONE     | 不做任何的转换，直接以该数据所属的数据类型进行存储。         |

## SQLite 亲和类型(Affinity)及类型名称

下表列出了当创建 SQLite3 表时可使用的各种数据类型名称，同时也显示了相应的亲和类型：

| 数据类型               | 亲和类型 |
| :--------------------- | :------- |
| INTINTEGER             | INTEGER  |
| SMALLINT               | INTEGER  |
| TINYINT                | INTEGER  |
| MEDIUMINT              | INTEGER  |
| BIGINT                 | INTEGER  |
| UNSIGNED               | INTEGER  |
| BIGINT                 | INTEGER  |
| INT2                   | INTEGER  |
| INT8                   | INTEGER  |
| CHARACTER(20)          | TEXT     |
| VARCHAR(255)           | TEXT     |
| VARYING CHARACTER(255) | TEXT     |
| NCHAR(55)              | TEXT     |
| NATIVE CHARACTER(70)   | TEXT     |
| TEXTCLOB               | TEXT     |
| NVARCHAR(100)          | TEXT     |
| BLOB                   | NONE     |
| no datatype specified  | BLOB     |
| REAL                   | REAL     |
| FLOAT                  | REAL     |
| DOUBLE PRECISION       | REAL     |
| DOUBLE                 | REAL     |
| NUMERIC                | NUMERIC  |
| DECIMAL(10,5)          | NUMERIC  |
| BOOLEAN                | NUMERIC  |
| DATETIME               | NUMERIC  |
| DATE                   | NUMERIC  |

## Boolean 数据类型

`SQLite` 没有单独的 `Boolean` 存储类。相反，布尔值被存储为整数 0(false) 和 1(true).

注: MySQL当中同样也没有布尔类型的数据.

## Date 与 Time 数据类型

`SQLite` 没有一个单独的用于存储日期和/或时间的存储类，但 `SQLite` 能够把日期和时间存储为 `TEXT`、`REAL` 或 `INTEGER` 值。

| 存储类  | 日期格式                                                     |
| :------ | :----------------------------------------------------------- |
| TEXT    | 格式为 "YYYY-MM-DD HH:MM:SS.SSS" 的日期。                    |
| REAL    | 从公元前 4714 年 11 月 24 日格林尼治时间的正午开始算起的天数。 |
| INTEGER | 从 1970-01-01 00:00:00 UTC 算起的秒数。                      |

您可以以任何上述格式来存储日期和时间，并且可以使用内置的日期和时间函数来自由转换不同格式。

---

## Python中的使用

在使用上和MySQL_connector类似

```python
import sqlite3 as sqlite
# 连接, 如果数据库尚未创建, 会自动创建
sql = sqlite.connect('test.db')
# 创建游标
cursor = sql.cursor()
# 事务, 和MySQL类似, 执行是在事务中执行的
sql.commit()
# 关闭, 关闭时, 未执行的事务会自动提交?
cursor.close()
sql.close()
# 其他诸如, 数据查询返回, 和MySQL都是类似的
cursor.fetchall()
```

**需要注意的是**: 批量插入数据中的`cmd`语句占位符号使用的 `"?"`, 而不是像MySQL使用的是 `"%s"`

![position](https://p0.meituan.net/dpplatform/21b706bf41ac7a5ac6fbb25fe6938d966401.png)

```python
cursor.executemany(cmd, data_list)
```

## 更新数据

注意语法上的差异: 

*这里的更新数据指的是, 数据存在则更新, 不存在则写入.*

这项功能再MySQL上是使用 `ON DUPLICATE KEY UPDATE`(当然也可以使用select返回结果判断间接实现)

> # How to do INSERT with an UPDATE on duplicate using SQLite
>
> How to do an UPDATE from an INSERT with SQLite when there is a duplicate key value.
>
> With SQLite you cannot do the simple MySQL INSERT on duplicate key UPDATE:
>
> INSERT INTO `table` **(**id, name, price, quantity**)** VALUES**(**1, 'test', '2.50', 164**)**
>
> ​    ON DUPLICATE KEY UPDATE `quantity` = 164, `price` = '2.50'
>
> Instead, you have to do what is called an [upsert](https://www.sqlite.org/draft/lang_upsert.html).
>
> The concept is very similar to the MySQL example above. The differences being you have to specify which column is the indexed/key column (unique) and then state the DO UPDATE:
>
> INSERT INTO users**(**username,score**)** VALUES**(**'Johnny', 388**)** 
>
> ON CONFLICT**(**username**)** **DO** UPDATE SET score = '388';
>
> 注: 这里可以插入更新的数据, 同时可以更新指定的数据
>
> Had you need to update multiple columns simply separate each update instance with a comma:
>
> **DO** UPDATE SET score = '388', rating = 'A';
>
> From the example if a row already has the value “Johnny” in the *name* column then the *score* column value will be updated to be 388.
