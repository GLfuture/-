#  MySQL

## MySQL配置

+ 启动服务
  + cmd中输入services.msc打开服务窗口
  + 使用管理员打开cmd---> net start/stop 服务名称
+ 登录登出

## SQL

结构化查询语言

### 基本语法

+ SQL语句可以多行书写，每句以;结尾
+ 可用空格和缩进来增强可读写
+ 不区分大小写，但关键词还是要用大写
+ 注释 --或#或/**/(多行)

### 分类

+ DDL数据定义语言，用来定义数据库对象
+ DML数据操作语言，增删改查
+ DQL数据查询语言，用来查询数据库中表
+ DCL数据控制语言，定义数据库的访问权限及安全级别

### DDL

#### 操作数据库

>  C(Creat):创建

+ `create database 数据库名称`   		创建数据库
+ `create database if not exists 数据库名称`   判断是否存在再创建
+ `create database 数据库名称 character set 字符集`指定数据库的字符集
+  `create database if not exists 数据库名称 character set 字符集`    注意顺序

>  R(Retrieve):查询

+ `show databases`		查询所有数据库的名称
+ `show create database 数据库名称` 	查询某个数据库创建时使用的字符集 	

> U(Update):修改

​				`alter database 数据库名称 character set 字符集名称`     	修改数据库字符集

> D(Delete):删除

+ `drop database 数据库名称`    	删除数据库
+ `drop database if exists 数据库名称`  		判断存在再删除

> 使用数据库

+ `select database()`      	查询 当前正在使用的数据库
+ `use 数据库名称` 			改变使用的数据库

#### 操作表

> C(Creat):创建

```sql
create table 表名(
		列名1  数据类型1,
		列名2  数据类型2,
		列名3  数据类型3,
		...
		列名n  数据类型
) 
```

创建一个表

`create table 表名 like 被复制的表名`复制一个表

sql 数据类型

1. int 整形
2. double(小数点后最大位数,显示的位数) 浮点数形
3. date 日期,只包含年月日	yyyy-MM-dd
4. datetimeL 日期,还包含时分秒     yyyy-MM-dd HH:mm:ss
5. timestamp  时间戳 ,不赋值或者赋值为null的话默认使用当前系统时间自动赋值
6. varchar(定义最大长度)  字符串

> R(Retrieve):查询

+ `show tables`			查询所在数据库中所有表的名称
+ `desc 表名`		查询表的结构

> U(Update):修改

1. 修改表名  `alter table 表名  rename to 新表名`
2. 修改表的字符集	`alter table 表名 character set`
3. 添加一列   `alter table 表名 add 列名 数据类型`
4. 修改列的名称 类型  
   + `alter table 表名 change 列名 新列名 新数据类型`
   + `alter table 表名 modify 列名 新数据类型`
5. 删除列  `alter table 表名 drop 列名`

> D(Delete):删除

+ `drop table 表名`      删除表
+ `drop table 表名 if exists`   加个判断

### DML

> 添加数据

`insert into 表名 (列名1,列名2,...列名n) values (值1,值2,...值n)`

**如果表明后不定义列名,则给所有列加值**

> 删除数据

`delete from 表名 where 条件`

+ **如果不加条件，则删除表中所有数据**
+ 要删除所有数据则可以`truncate table 表名`

> 修改数据

`update 表名 set 列名1 = 值1,列名2 = 值2, ... 列名3 = 值3 where 条件`

如果不加任何条件则会将表中的所有数据修改

### DQL

#### 语法

```sql
select
	字段列表
from
	表明列表
where
	条件列表
group by
	分组字段
having
	分组后的条件
order by
	排序
limit
	分页限定
```

#### 基础查询

+ 去重：distinct 

  ```sql
  select distinct score
  from student
  ```

+ 计算列

  + 一般可以使用四则运算计算一些列的值
  + ifnull(字段,替换值),如果字段为null则替换为替换值

+ as 可省略

  ```sql
  select name,age,score,age + score as 总分
  from student
  ```

  

#### 条件查询(where)

+ 基本与算符
+ `字段 between... and ...`     一个区间
+ `字段 in()`   集合
+ `字段 is (not) null` 判断是否(不)为null
+ 模糊查询:`字段 like 占位符` 占位符_代表任意一个字符，%代表任意多个字符。

#### 排序查询(order by)

+ `order by 字段1  排序方式1 , 字段2 排序方式2,...字段n  排序方式n`;
+ 排序方式
  + ASC，升序，默认
  + DESC，降序
+ 有多个排序条件，只有前面的条件一样时才按后面的规则排

```sql
select *
from student
order by score desc ,ID asc ;
```



#### 聚合函数

+ count:计算个数
+ max : 计算最小值
+ min: 计算最小值
+ sum: 求和
+ avg: 计算平局值
+ group_concat：联合

**聚合函数会排除null的值**

#### 分组查询(group by)



```sql
select sex, avg(score)
from student
group by sex
having count(id) > 2;
```

+ where  和 having 的区别
  + where 用在分组前 having用在分组后
  + where不能用聚合函数 having可以

#### 分页查询

`limit 开始的的索引, 查询几条数据 `

开始的索引 = (当前页码 - 1) * 查询的数据

----

### 约束

对表中的元素进行限定，保证元素的正确性有效性完整性，防止非法数据入库

> 非空约束(Not null)

+ 创建表时添加约束条件

  ```sql
  create table 表名(
  		列名1  数据类型1  Not null,
  		列名2  数据类型2  Not null,
  		列名3  数据类型3  Not null,
  		...
  		列名n  数据类型
  ) 
  ```

+ 创建完表添加(修改列的类型)

+ 删除非空约束(修改表的类型)

> 唯一约束(unique)

```sql
create table 表名(
		列名1  数据类型1  unique,
		列名2  数据类型2  unique,
		列名3  数据类型3  unique,
		...
		列名n  数据类型
) 
```

+ 添加同上(在表中没重复数据的时候才能添加)
+ 删除`alter table 表名 drop index 列名`

> 主键约束

表示非空且唯一，且一张表只有一个主键约束

```sql
create table 表名(
		列名1  数据类型1  primary key auto_increment,
		列名2  数据类型2,
		列名3  数据类型3
		...
		列名n  数据类型
) 
```

删除`alter table 表名 drop primary key`

> 外键约束

添加外键

```sql
create table 表名(
		列名  数据类型,
    	...
		constraint 外键名称 foreign key (外键列名称) references 主表名称(主键名称)
) 
```

删除外键`alter table 表名 drop foreign key 外键名`

添加外键`alter table 表名 add constraint 外键名称 foreign key (外键列名称) reference 主表名称(主表列名称)`

**注意**：主表设置的外键的必须是子表的主键！！

### 数据库的设计

+ 多表之间的关系
  + 一对一
  + 一对多:多的一方建立外键，指向一的一方
  + 多对多:要使用第三张至少包含两个字段的表实现，这些字段分别指向两张表的主键
+ 数据库的设计范式
  + 第一范式(1NF):每一列都是不可分割的原子数据项
  + 第二范式(2NF):所有非主属性完全依赖于码
    + 函数依赖:A-->B,如果通过A属性(组)能确定唯一一个B的值，则称B依赖于A
    + 完全函数依赖，部分函数依赖，传递函数依赖
    + 码，如果一个属性(组)被其他所有属性完全依赖，则称这个属性(组)为该表的码
    + 主属性:码属性组中的所有属性
    + 非主属性:除码属性的所有属性
  + 第三范式(3NF):消除传递依赖

### 数据库的备份与还原

+ 命令行
  + 备份: mysqldump -u用户名 -p密码 数据库名称
  + 还原:登录数据库，创建数据库，使用数据库，`source 文件路径`

### 多表查询

+ 内连接查询：
  + 隐式内连接:使用where消除无用数据
  + 显式内连接: `select  字段列表 from 表名1  join 表名2 on 条件`
+ 外连接查询:
  + `select  字段列表 from 表名1 left join 表名2 on 条件`
+ 子查询:



### 事务

+ 基本介绍
  + 概念:如果一个包含多个步骤的业务操作被事务管理，那么这些 操作要么同时成功要么同时失败
  + 操作:
    + 开启事务(start transaction)
    + 回滚(roll back)
    + 提交(committed)
+ 事务的四大特征
  + 原子性:是不可分割的最小操作单位，要么同时成功，要么同时失败
  + 持久性:当事务提交或回滚后，数据库会持久化保存数据
  + 隔离性:多个事务之间，相互独立
  + 一致性，事务操作后，数据总量不变

### DCL

+ 管理用户(mysql库中的user表)
  + 添加用户:`create user '用户名'@'主机名' identified by '密码'`
  + 删除用户`drop user '用户名'@'主机名'`
  + 查询用户:`select * from user`
  + 修改密码
+ 权限管理
  + 查询权限:`show grants for '用户名'@'主机名'`
  + 授予权限`grant 权限列表 on 数据库名.表名 to '用户名'@’主机名`
  + 撤销权限:`revoke 权限列表 on 数据库名.表名 from '用户名'@’主机名` 
    授权用户(全部权限)
    grant all privileges on 表名 to 'user1'@'ip地址' identified by '密码'

### mysql远程登录命令

```
mysql -u root -h ip -p
```

