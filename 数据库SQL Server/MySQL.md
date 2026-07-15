==id 一般是自增主键，不应该出现在 INSERT 语句中，让数据库自动生成即可。==
-  命令行启动
```shell
mysql -u root -p    //登录启动
mysql -h[ip地址] -P[端口号] -uroot -p     //远程访问数据库

SHOW VARIABLES LIKE 'datadir';  //查看data目录

net start  //查看当前已经启动的Windows服务

net start mysql
net stop mysql

mysqladmin -u root password 1234

exit       //退出mysql

create database <库名>;


- 操作语法

-查询所有数据库
show databases;

-查询当前数据库
select database();

--使用/切换数据库
use数据库名;

--创建数据库
create database [if not exists] 数据库名 [default charset utf8mb4];

删除数据库
drop database [if exists]数据库名;


```

![](图片/数据库SQL%20Server/MySQL/IMG-20260618143738149.png)
- 创建表
```sql
create table tablename(
字段1名 字段类型 [约束] [comment '字段1注释'],
字段2名 字段类型 [约束] [comment '字段2注释']
)[comment '表的注释'];

create table user(
id int primaty key auto_increment comment'ID,唯一标识', -- 这是主键 auto_increment自动增长
username varchar(50) not null unique comment '用户名',
name varchar(10) comment '姓名'，
age int comment '年龄',
Igender default '男' char(1) comment'性别"
)comment "用户信息表"

```
![](图片/数据库SQL%20Server/MySQL/IMG-20260618145845213.png)

- 数值类型 ![](图片/数据库SQL%20Server/MySQL/IMG-20260618150632115.png) ![](图片/数据库SQL%20Server/MySQL/IMG-20260618150729327.png)

- 字符串类型
![](图片/数据库SQL%20Server/MySQL/IMG-20260618151030799.png)
- 日期时间类型
![](图片/数据库SQL%20Server/MySQL/IMG-20260618151149400.png)
- ==表结构==的查询、修改、删除相关语法如下:
```sql
show tables;  -- 查询当前数据库的所有表

desc 表名;   -- 查询表结构

show create table 表名;  -- 查询建表语句

alter table 表名 add 字段名 类型(长度) [comment 注释] [约束];-- 添加字段(新一列)

alter table 表名 modify 字段名 新数据类型(长度);-- 修改字段类型(一列的数据类型)

alter table 表名 change 旧字段名 新字段名 类型(长度) [comment 注释] [约束];-- 修改字段名与字段类型

alter table 表名 drop column 字段名;-- 删除字段

alter table 表名 rename to 新表名;-- 修改表名

drop table [if exists] 表名;-- 删除表，表中数据也会删除

show tables;
desc emp;
show create table emp;
alter table emp add qq varchar(13) comment 'QQ号码';
alter table emp modify qq varchar(15);
alter table emp change qq qq_number varchar(20) comment '超级QQ号';
alter table emp drop column qq_number;
alter table emp rename to emppp;
drop table emppp;


```

DML（数据操作语言）
```sql
DML-insert
-- 指定字段添加数据
insert into 表名(字段名1，字段名2）values（值1,值2）

-- 全部字段添加数据
insert into 表名 values (值1，值2，·..);

-- 批量添加数据（指定字段）
insert into 表名 (字段名1,字段名2) values (值1，值2),(值1，值2);

-- 批量添加数据（全部字段）
insert into 表名 values (值1,值2,...),(值1,值2,...);

DML-update
-- 修改数据
update 表名 set 字段名1=值1,字段名2=值2,... [where 条件];

update emp set name='张三' where id = 1;

DML-delete 
--删除某条数据
delete from 表名 where 条件
delete from emp where id = 1;
```

DQL（查询）
```sql
DQL-基本查询

-- 查询多个多个字段
select 字段1，字段2，字段3 from 表名;      表中这几列的所有信息

-- 查询所有字段
select * from 表名;

-- 为查询字段设置别名，as 可以省略
select 字段1[as 别名1] from 表名;        查询结果的列名的别名

-- 去除重复记录
select distinct 字段 from 表名;          会直接对列去重

DQL-条件查询
select 字段 from 表名 where 条件列表;
```
![](图片/数据库SQL%20Server/MySQL/IMG-20260618155407392.png)
```sql
DQL-分组查询

-- 聚合函数  
-- 所有聚合函数不参与null的统计
select count(字段 * 常量) from 表名;

select avg(字段) from 表名;

select min() from 表名;

select max() from 表名;

select sum() from 表名;

-- 分组查询
select 字段列表 from 表名 [where 条件列表] group by 分组字段名 [having 分组后过滤条件]

--●where与having的区别：
--1.执行时机不同：where是分组之前进行过滤，不满足where条件，不参与分组；而having是分组之后对结果进行过滤。
--2．判断条件不同：where不能对聚合函数进行判断，而having可以。

-- select 后面一版是 分组字段 + 聚合函数
select gender,count(*) from emp group by gender;

```
![](图片/数据库SQL%20Server/MySQL/IMG-20260618160908656.png)

- DQL-排序查询
- ![](图片/数据库SQL%20Server/MySQL/IMG-20260618161007048.png)
- ![](图片/数据库SQL%20Server/MySQL/IMG-20260618161421466.png)
