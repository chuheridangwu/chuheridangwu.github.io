---
title: SQLite数据库基常见操作
date: 2018-07-23 14:41:55
tags: [SQLite]
categories: 数据库
---

**SQL语句两个注意点：**
1、语句不区分大小写
2、每条语句以`;`号结尾

**关键字：**
select/insert/update/delete/from/create/where/desc/order/by/group/table/alter/view/index

### DDL - 数据定义命令
#### 创建表 CREATE

**创建单张表**

```
create table if not exists 表名（字段名称 字段类型 字段属性,字段名称 字段类型）
create table if not exists t_student (id integer primary key autoincrement , name text, age integer, score real);
```
`if not exists` : 如果没有这张表，才进行创建
`primary key` : 设为主键
`autoincrement` : 主键自增

<!-- more -->
**创建关联表**

```
create table if not exists 表名（字段名称 字段类型 字段属性,字段名称 字段类型）CONSTRAINT 别名<关联起的别名> 字段名称 FOREIGN KEY (当前表需要关联的字段名称) REFERENCES 关联表(关联表的字段名称)
create table if not exists t_student (id integer primary key,age integer default 1,class_id integer, CONSTRAINT fk_t_student_class_id_t_class_class_id FOREIGN KEY (class_id) REFERENCES t_class(id));
```
`default` : 表示默认值
`CONSTRAINT`: 关联
`FOREIGN KEY`: 当前表关联的字段名
`REFERENCES` : 关联表的字段名

**字段简单约束**
`not null`: 规定字段的值不能为null
`unique`: 规定字段的值必须唯一
`default`: 指定字段的默认值


#### 更新表 ALTER
SQLite 的 ALTER TABLE 命令不通过执行一个完整的转储和数据的重载来修改已有的表。您可以使用 ALTER TABLE 语句重命名表，使用 ALTER TABLE 语句还可以在已有的表中添加额外的列。
在 SQLite 中，**除了重命名表和在已有的表中添加列，ALTER TABLE 命令不支持其他操作。**

**在表中添加列 `add`**

```
alter table t_student add firstName text;
```

**修改表名**

```
alter table t_student rename to new_t_student;
```

#### 删除表 drop
**删除表**

```
drop table t_student
```

**删除表中的列 `SQLite`除外的其他数据可以使用**

```
alter table t_student drop column name1;
```
### DML - 数据操作命令
#### 插入记录 insert
**指定列名，插入单个数据**

```
insert into 表名(字段名称,字段名称) values (值,值);
insert into t_student(name,age) values ('aaa',14);

```

#### 修改记录 update
**修改所有行的数据**

```
update 表名 set 字段名称 = 19
update t_student set age = 19, name = 'jack'
```

**根据条件修改行数据**

```
 where 字段 = 某个值
 update  t_student set age = 19 where age=120
 
 where 字段 is 某个值
 update  t_student set age = 19 where age is 120
 
 where 字段 != 某个值
 update  t_student set age = 19 where age != 120
 
 where 字段 isnot 某个值
 update  t_student set age = 19 where age isnot 120
 
 where 字段 > 某个值
 update  t_student set age = 19 where age > 120
 
 where 字段1= 某个值 and 字段2 > 某个值
 update  t_student set age = 19 where age > 20 and name = jack
 
 where 字段1= 某个值 or 字段2 > 某个值
 update  t_student set age = 19 where age > 120 or name = jack
```

#### 删除记录 delete

**删除表中所有数据**
```
delete from 表名 
delete from t_student 
```
**根据条件删除单行数据，条件同更新一样**


### DQL - 数据查询命令
#### 查询 select

**所有当前表所有字段**

```
select  *from t_student
```

**查询当前字段**

```
select 字段名称 from 表名 表的别名<写了更方便，不写字段名>;
select s.age from t_student s;
```
`s`: 表的别名
`s.age`: 表字段

**对查询结果进行排序 `asc` 升序 `desc` 降序**

```
select *from 表名 order by 表字段 desc, 表字段 asc
select  *from t_student s order by  s.tag desc ,s.age asc
```

**根据条件进行查询**

```
select *from 表名 where 条件 order by 表字段 desc, 表字段 asc
select  *from t_student s where s.age > 10 order by  s.tag desc ,s.age asc
```

**分页查询**

```
 select *from 表名 limit 数值1, 数据长度; 跳过前四个数据，然后取8条记录
 select *from t_student limit 4, 8;
```

**嵌套查询**

```
select 字段名称 from 表名 where 条件 = (select 字段名 from 表名 条件)
select c.name,c.id from t_class c where c.id=(select s.class_id from t_student s where s.name = 'test')
```

**多表关联查询**

```
select 字段名称 from 表名, 表名 where 条件 
select s.id,s.name from t_class c,t_student s where c.id = s.class_id and age > 12 and s.name = 'test';
```

 **当前查询到的数据数量**
 
 ```
 select count(*) from 表名；
 select count(*) from t_student；
 ```
 
## 参考链接
[SQLite 教程](http://www.runoob.com/sqlite/sqlite-alter-command.html)



