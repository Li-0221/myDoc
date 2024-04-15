# 开启远程连接

1. 打开命令提示符（cmd）。
2. 输入 `mysql -u root -p` 并按回车键。
3. 输入密码并按回车键。
4. 输入 `CREATE USER 'root'@'%' IDENTIFIED BY '19990221li';` 并按回车键，这将创建一个具有相应权限的用户。
5. 输入 `GRANT ALL PRIVILEGES ON _._ TO 'root'@'%';` 并按回车键，这将授予用户所有权限。
6. 输入 `FLUSH PRIVILEGES;` 并按回车键，以刷新权限。
7. 现在，你已经在 MySQL 中创建了一个允许来自任何主机的 root 用户，并赋予了所有权限。

# MySQL

## SQL 语句

> sql 语句之间要用 ; 分隔开

## 1.增删改查

> 基础操作

```sql
-- 共有三列数据 id name age

select * from table_name;
select name from table_name;
select name,age from table_name;

insert into table_name(name,age) values('张三',18);
insert into table_name values(1,'李四',19);

update table_name set name='王五' where id=1;

delete from table_name where id=1;
```

使用 set 插入

```sql
insert into table_name set name='赵六',age=18;
```

## 2.where、and、or

> where 一般用于限定 SQL 语句的条件  
> and&or 可以在 where 中把多个条件结合起来

<img src="https://s1.ax1x.com/2022/04/19/L0mk3n.png" alt="set插入字段" />

## 3.order by

> order by 用于根据指定的列对结果集进行排序

默认是升序，如果要使用降序，可以使用 desc 关键字

```sql
select * from table_name order by id;
select * from table_name order by id desc;
```

多重排序：有多个排序的条件时，由逗号分隔

```sql
select * from table_name order by id desc,age asc;
```

## 4.count(\*)&as

> count(\*)用于返回查询结果的条数

```sql
select count(*) from table_name where age=18;
```

> as：为列设置别名

```sql
select count(*) as total from table_name;
select name as 用户名,age as 年龄 from table_name;
```

## 5.limt&like

> 使用于分页查询等应用场景

下列代码表示从第 5 个数据开始，查 6 个数据

```sql
select * from table_name limit 4,6;
```

limit 于 like 结合使用

```sql
select * from table_name where name like '%张%' limit 4,6;
select * from (select * from table_name limit 1,7) a where a.name like '%张%';
```
