# MySQL 基本操作

# MySQL

[TOC]

本机MySQL 默认不会开机自启， 

账号：root 

密码：rootroot 

------

## 注释

- 单行注释： --
- 多行注释 ：/* */

## SQL分类

- DDL ：Data Definition Language 数据定义语言：create、drop、alert等
- DML：Data Manipulation language 数据操作语言： insert、update、delete等
- DQL：Data Query Language 数据查询语言：select、where等
- DCL：Data Control Language 数据控制语言（定义数据库访问权限和安全级别以及创建用户）：GRANT、REVOKE等

## 操作数据：CRUD

 1. C(Create):创建

 2. R(Retrieve):查询

 3. U(Update):修改

    4. D(Delete):删除

    ```sql
    /*-----------------------数据库操作----------------------*/
    /*创建数据库*/
    create database 数据库名称;
    /*创建数据库，如果不存在的话*/
    create database if is not exists 数据库名称;
    /*创建数据库 并设置字符集*/
    create database 数据库名称 character set GBK;
    
    /*查询所有数据库的名称*/
    show databases;
    /*查询创建数据库的语法*/
    show create database mysql;
    /*查询某个数据库的字符集*/
    show create database 数据库名称;
    
    /*修改数据库的字符集*/
    alter database 数据库名称 character set 字符集;
    
    /*删除数据库*/
    drop database 数据库名称;
    drop database 数据库名称 if exists;
    
    /*查询当前正在使用的数据库*/
    seletct database();
    /*使用数据库*/
    use 数据库名称
    
    /*-----------------------表操作----------------------*/
    /*创建表*/
    create table 表名(列名1 数据类型,列名2 数据类型... );
    /*查询数据库中所有表的名称*/
    show tables;
    /*查询表结构*/
    desc 表名;
    /*复制表*/
    create table 表名 like 被复制的表;
    
    /*修改表名*/
    alter table 表名 rename to 新的表名;
    /*修改表的字符集*/
    alter table 表名 character set 字符集;
    /*添加一列*/
    alter table 表名 add 列名 数据类型;
    /*修改列名称 类型*/
    alter table 表名 change  列名 新列名;
    alter table 表名 modify  列名 新数据类型;
    alter table 表名 change  列名 新列名 新数据类型;
    /*删除列*/
    alter table 表名 drop 列名;
    
    /*删除表*/
    drop table 表名 if exists;
    
    
    ```

## MySql支持的数据类型

## 基本增删改查

```sql

/*-----------------------表中数据操作----------------------*/
/*添加 如不定义列名,则默认给所有列添加值*/
insert into 表名 (列名1,列名2....) values (值1,值2,...)

/*删除 如无条件，则删除全部*/
delete from 表名 where [where条件]
/*删除表，再创建一个一样的新表 */
truncate table 表名

/*修改数据*/
update 表名 set 列名1=值1,列名2=值2.... [where条件]

/*查询排序 升序(默认)：asc  降序 desc*/
select * from 表名 order by 列名 排序方式
```

查询：

1. 排序

   - `升序(默认)：asc` 

   - `降序：desc`

     ```sql
     select * from 表名 order by 列名1 排序方式1,列名2 排序方式2 ... [where条件]
     ```

2. 聚合函数 

   - `count ：个数`

   - `max：最大值`

   - `min：最小值`

   - `sum：和`

   - `avg：平均值`

     ```sql
     select count(*) from 表名 --查询条数，只要有一个值不为空就算一跳记录
     select count(列名) from 表名  --会排除null值
     select count(列名 ifnull(列名,默认值)) from 表名 --不排除null值，设置null时的默认值
     ```

3. 分组 `group by`

   `group by 分组字段`

   `having 分组后的限定条件`

   ```sql
   select * from 表名 [where 条件] group by 分组条件 having [having 限定条件]; 
   ```

   where  和 having 的区别 ：

    1. [where 条件] 分组前限定，并不可用聚合函数判断

    2. [having 限定条件] 分组后限定，可与聚合函数一起判断

       例: 查询学生表中，男女生数学成绩的平局值，要求：参与分组分数不得小于70人，且人数不少于2人

       ```sql
       select sex,avg(math),count(id)  from student where math>70 group by sex having count(id)>2;
       ```

4. 分页 `limit`

   `limit 开始的索引，每页的条数`

   ```sql
   // 每页3条数据，查询第一页的数据
   select * from student limit 0,3
   ```

5. 去重`distinct`

   ```sql
   //id去重
   select distinct id from student 
   //多个字段去重，只有多个字段完全一样才会被去重
   ```

6. 模糊查询

   ```sql
   select * from student where name like "a%";
   //第二个字是a的
   select * from student where name like "_a%";
   //名字是三个字的
   select * from student where name like "___";
   //名字包含a的
   select * from student where name like "%a%";
   ```

7. 其他基础

   `>、<、>=、<=、=`

   `between..and`

   `in 集合`

   `like`

   `is null`

   `and 或者 &&`

   `or 或者 ||`

   `not 或者 !`

## 约束

例子：

```sql
CREATE TABLE stu(
    id INT PRIMARY KEY AUTO_INCREMENT,   --主键
    name VARCHAR(20) NOT NULL,   --非空
    phone VARCHAR(20) UNIQUE --唯一
    );
```



1. 主键:primary key   (非空且唯一，一张表只能有一个字段为主键)

   ```sql
   -- 添加主键
   ALTER TABLE stu MODIFY id INT PRIMARY KEY;
   -- 主键 无法用一下语句删除
   ALTER TABLE stu MODIFY id INT;
   -- 主键删除
   ALTER TABLE stu DROP PRIMARY KEY;
   -- 删除主键自增长
   ALTER TABLE stu MODIFY id INT;
   ```

   

2. 非空约束:not null  (字段不能为空)

   ```sql
   -- 添加非空约束
   ALTER TABLE stu MODIFY name VARCHAR(20) NOT NULL; 
   -- 删除非空约束
   ALTER TABLE stu MODIFY name VARCHAR(20) ;
   ```

   

3. 唯一约束:unique   (字段值唯一，注意MySQL中 null可有多个)

   ```sql
   ALTER TABLE stu MODIFY phone VARCHAR(20) UNIQUE;
   -- 唯一约束 无法用一下语句删除
   ALTER TABLE stu MODIFY phone VARCHAR(20) ;
   -- 唯一约束正确的删除方式：
   ALTER TABLE stu DROP INDEX phone;
   ```

4. 外键约束:foreign key 

   语法： `CONSTRAINT [外键名称] FOREIGN KEY ([本表的字段]) REFERENCES 外键链接的表名([链接表名中的字段])`

   ```sql
   
   CREATE TABLE A(
    id INT PRIMARY KEY AUTO_INCREMENT
   );
   
   CREATE TABLE B(
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_A INT,
    CONSTRAINT a_b_id FOREIGN KEY (id_A) REFERENCES A(id)  -- 创建表时添加外键
   );
   
   
   -- 添加外键
   ALTER TABLE B ADD CONSTRAINT a_b_id FOREIGN KEY (id_A) REFERENCES A(id)
   -- 删除外键
   ALTER TABLE B DROP FOREIGN KEY a_b_id;
   
   -- 级联操作 
   -- 1.级联更新  ON UPDATE CASCADE 
   -- 2.级联删除  ON DELETE CASCADE 
   ALTER TABLE B ADD CONSTRAINT a_b_id FOREIGN KEY (id_A) REFERENCES A(id) ON UPDATE CASCADE ON DELETE CASCADE;
   
   ```

   

## 数据库设计的三大范式

**第一范式（1NF）**：数据库表的**每一列都是不可分割的原子数据项**

**第二范式（2NF）**：在第一范式的基础上，**实体的属性完全依赖于主关键字。所谓完全依赖是指不能存在仅依赖主关键字一部分的属性**

**第三范式（3NF）**：在2NF基础上，**任何非主**[**属性**](https://baike.baidu.com/item/%E5%B1%9E%E6%80%A7)**不依赖于其它非主属性（在2NF基础上消除传递依赖）**



## 数据库备份与还原

1. 备份： mysqldump -u用户名 -p密码  >  保存路径

   备份指定数据库： mysqldump -u用户名 -p密码 数据库名称  >  保存路径

2. 还原：
   1. 登陆数据库
   2. 创建数据库
   3. 使用数据库
   4. 执行备份文件，source 文件路径



## 多表查询

>  笛卡尔积：多表查询，清除无用数据

```sql
select * from table_a,table_b [where 条件]

-- 为表取别名
select * from table_a a,table_b b [where 条件]
```

1. 内连接：查询多表交集部分

   - **隐式内连接**：使用where 条件消除无用数据

     语法： `select 字段列表 from table1,table2 where table1.id = table2.id`

   - **显示内连接** ：

     语法： `select 字段列表 from table1 [inner] join table2 ON [条件]`  (inner 可省略)

2. 外链接

   - 左外连接：查询的是左表的全部信息，以及与右表其交集部分

     语法：`select 字段列表 from table1 left  [outer]  join table2 on [条件]`

   - 右外连接：查询的是右表的全部信息，以及与左表其交集部分

     语法：`select 字段列表 from table1 right  [outer]  join table2 on [条件]`

3. 子查询

   - 单行多列：作为条件使用运算符 >、<、=、>=、<=

   - 多行单列：作为条件运算符 IN

   - 多行多列：作为虚拟表进行多表联查

     ```sql
     select * from table1 t1,(select * from table2 [where]) t2 where table1.id = table2.id
     ```

     

## 事务

1. **事务概念**：如果一个包含多个步骤的业务操作，被事务管理，要么同时成功，要么同事失败。

2. **事务操作方式**：

   1. 开启事务：start transaction;
   2. 回滚：rollback;
   3. 提交：commit;

   ```sql
   -- 开启事务
   start transaction;
   -- 被事务管理的sql语句
   ...
   -- 执行通过
   commit;
   -- 出现问题，回滚事务  删除临时数据，回滚数据到开启事务之前
   rollback;
   
   ```

   

   > **关于数据提交需要注意的地方**：
   >
   > MySQL 中事务默认自动提交，Oracle 默认手动提交
   >
   > 自动提交：增删改语句会默认自动提交，使数据持久化
   >
   > 手动提交：开启事务的情况下，必须commit 手动提交，才能让数据持久化
   >
   > 
   >
   > 修改事务的默认提交方式：
   >
   > 查询默认提交方式：`select @@autocommit;` -- 1 自动提交 2 手动提交
   >
   > 修改默认提交方式：`set @@autocommit=0;`
   >
   >    *修改完成后每一条sql都需要手动更新*

3. **事务的四大特征**
   1. 原子性：不可分割的最小操作单位
   2. 持久性：事务一旦提交或者回滚，会持久化保存数据
   3. 隔离性：多个事务之间相互独立
   4. 一致性：事务操作前后数据总量不变

4. **事务的隔离级别**

   用来处理多个事务处理同一批数据引发的一些问题

   存在问题：

   1. 脏读：一个事务读到另一个事务中没有提交的数据
   2. 不可重复读(虚度)：同一个事务中，两次独到的数据不一样
   3. 幻读：一个事务操作(DML)数据表中所有的记录，另一个事务添加了一条数据，则第一个事务无法查询到自己的修改

   隔离级别：

   1. `read uncommitted` :读未提交

      *问题：脏读、不可重复读、幻读*

   2. `read committed` :读已提交(Oracle 默认)  (只有提交了数据另一个事务才能读到)

      *问题 : 不可重复读、幻读*

   3. `repeatable read` : 可重复读(MySQL 默认)

      *问题 : 幻读*

   4. `serializable`：串行化 (锁表)

      *解决所有存在的问题*

   **注意：隔离级别 安全性越来越高，效率越来越低**

   

   关于数据库隔离级别设置：

   ```sql
   -- 查询事务的隔离级别
   select @@tx_isolation;
   -- 设置隔离级别
   set global transaction isolation level 级别字符串;
   ```

## 数据库管理员

1. 管理用户

   1. 添加用户

      语法：`create user '用户名'@'主机名' identified by '密码';`

   2. 删除用户

      语法：`drop user '用户名'@'主机名';`

   3. 修改用户密码

      语法：`update user set password = password('新密码') where user ='用户名'`

      或者：`set password for '用户名'@'主机名' = password('新密码');`

   4. 查询用户

      1. 切换到mysql数据库

         `use mysql;`

      2. 查询user 表

         `select * from user;`

      

2. 管理权限

   1. 查询权限：`show grants for '用户名'@'主机名';`

   2. 授予权限：`grant 权限列表 on 数据库.表名 to  '用户名'@'主机名';`

      授予所有权限：`grant ALL on *.* to  '用户名'@'主机名';`

   3. 撤销权限： `revoke 权限列表 on 数据库.表名 from '用户名'@'主机名';`