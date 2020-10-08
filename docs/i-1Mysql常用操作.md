# Mysql

## 常用操作

1. 创建数据库

   ```
   CREATE DATABASE mydb CHARACTER SET utf8 COLLATE utf8_general_ci; 
   ```
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

   

