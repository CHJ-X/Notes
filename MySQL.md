# MySQL

登录用户：

```mysql
mysql -u `username` -p
```

展示数据库和表：

```mysql
show xxx
```

选择库：

```mysql
use xxx
```

看表的详细：

```mysql
desc xxx;
```

选取所有列：

```mysql
SELECT * FROM xxx;
```

修改列信息基本都是用ALTER

-------------

把列数据替换为行号（更新列数据值为行号）

```mysql
SET @num = 0;
UPDATE `tablename` SET rowname  = (@num := @num + 1);
```

将自增主键设置为1

```mysql
ALTER TABLE table_name AUTO_INCREMENT = 1;
```

用python输入数据的时候记得要 _connect.commit()_
