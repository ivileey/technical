# Mysql

## 常用操作

1. 创建数据库

   ```
   CREATE DATABASE mydb CHARACTER SET utf8 COLLATE utf8_general_ci; 
   ```
- `limit y` 分句表示: 读取 y 条数据
- `limit x, y` 分句表示: 跳过 x 条数据，读取 y 条数据
- `limit y offset x` 分句表示: 跳过 x 条数据，读取 y 条数据

## 事务

1.  模仿数据库的事务的4种隔离级别

   ```
   SELECT @@autocommit; //查询事务提交状态
   set autocommit=0; //取消自动提交
   create database tran; //创建数据库
   use tran; //使用数据库
   create table student(id int primarykey,name varchar(10)) engine=innodb default charset=utf8; //创建表
   insert into student values(1,'zhangsan'); //使用表
   commit;//提交数据
   set session transaction isolation level read uncommitted;//将隔离级别设置成读未提交
   start transaction;//开启事务
   update student set name='msb';
   rollback;//回滚
   set session transaction isolation level read committed;//修改隔离级别为读取已提交
   set session transaction isolation level repeatable read;//
   
   ```

2. 加锁

   ```
   select * from student where id = 1 lock in share mode;
   select * from student where id = 1 for update;
   ```

   


## 查询

>  select指定的字段要么就要包含在Group By语句的后面，作为分组的依据；要么就要被包含在聚合函数中。

### 1.sql 课程成绩表 grade 字段 class sid score name

- 找出每门课的最高分的学生.

```
select b.name, b.class, b.score 
from (select subject, max(score) m from grade group by subject) t, grade b
where t.subject = b.subject and t.m = b.score
```

- 找出每门课最高分学生和最低分学生

```
select b.* from (select class,max(score) m from grade GROUP BY class) t,grade b where t.class = b.class and t.m=b.score UNION

select b.* from (select class,min(score) m from grade GROUP BY subject) t,grade b where t.class = b.class and t.m = b.score
```



1.  使用外连接的时候，注意是大表在前。

2.  ① select * from table limit 2,1;         

//含义是跳过2条取出1条数据，limit后面是从第2条开始读，读取1条信息，即读取第3条数据



② select * from table limit 2 offset 1;   

//含义是从第1条（不包括）数据开始取出2条数据，limit后面跟的是2条数据，offset后面是从第1条开始读取，即读取第2,3条

 