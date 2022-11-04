# MySQL翻译系列- executemany()方法



### 10.5.5 MySQLCursor.executemany() Method

Syntax:

```python
cursor.executemany(operation, seq_of_params)
```

This method prepares a database `operation` (query or command) and executes it against all parameter sequences or mappings found in the sequence `seq_of_params`.

这一方法用于查询或者执行命令操作, 假如两个组参数之间一一对应.

Note

注意事项

In Python, a tuple containing a single value must include a comma. For example, *('abc')* is evaluated as a scalar while *('abc',)* is evaluated as a tuple.

在python中, 一个元组包含包含单个值时, 必须包含逗号. 例如('abc'), 必须是('abc',), 这才会被视作元组.

In most cases, the `executemany()` method iterates through the sequence of parameters, each time passing the current parameters to the `execute()` method.

在大部分的案例中, executemany()通过迭代方式将参数传递给execute()(来实现执行).

注: 简而言之, 就是executemany(), 实际上是在execute()上的进一步的封装.

An optimization is applied for inserts: The data values given by the parameter sequences are batched using multiple-row syntax. The following example inserts three records:

在插入数据时的优化.

```python
data = [
  ('Jane', date(2005, 2, 12)),
  ('Joe', date(2006, 5, 23)),
  ('John', date(2010, 10, 3)),
]
stmt = "INSERT INTO employees (first_name, hire_date) VALUES (%s, %s)"
cursor.executemany(stmt, data)
```

For the preceding example, the [`INSERT`](https://dev.mysql.com/doc/refman/8.0/en/insert.html) statement sent to MySQL is:

实际上是将上述的内容拼成一个sql语句.

```sql
INSERT INTO employees (first_name, hire_date)
VALUES ('Jane', '2005-02-12'), ('Joe', '2006-05-23'), ('John', '2010-10-03')
```

With the `executemany()` method, it is not possible to specify multiple statements to execute in the `operation` argument. Doing so raises an `InternalError` exception. Consider using `execute()` with `multi=True` instead.

使用' executemany() '方法, 不可能在' operation '参数中指定要执行的多条语句. 这样做会引发一个' InternalError '异常. 考虑使用' execute() '和' multi=True '代替.

注: 这里的意思时, 在执行命令时, 不能将多条语句放在一起同时执行, 如创建多张表时, 分开创建, 或者使用multi=True, 表示改语言包含托条命令.

```mysql
create table test_a();
create table test_b();
```

```python
cmd = '''
    create table test_a();
    create table test_b();
'''
db.execute(cmd, multi=True)
# 不能直接执行, 注意这个问题db.execute(cmd), executemany(cmd) 
```

