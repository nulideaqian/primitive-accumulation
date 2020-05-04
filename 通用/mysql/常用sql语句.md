# 常用sql语句

## 数据库相关

## 表相关

```mysql
-- 查看建表语句
show create table <name>;
-- 修改表名
alter table <old_name> rename to <new_name>;
-- 添加列
alter table <name> add column <column_name> varchar(20);
-- 删除列
alter table <name> drop column <column_name>;
-- 修改列名
alter table <name> change <old_column_name> <new_column_name> int;
-- 修改列属性
alter table <name> modify <column_name> varchar(255);
```

