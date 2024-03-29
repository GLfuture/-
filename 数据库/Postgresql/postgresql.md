### postgresql

#### 登录

psql会先进入psql控制台，再进入数据库

```
psql -h 服务器 -U 用户名 -d 数据库 -p 端口
psql //默认是系统当前用户
```

默认情况下postgresql的超级用户为postgres

使用docker并创建kong用户时，需要先进入kong用户进入控制台，执行下列命令创建超级用户

```
create user postgres superuser;
create user root superuser;
```

#### 退出

```
exit //退出数据库
\q //退出psql控制台
```

#### 修改用户密码

```
alter user postgres with password 'postgres';
```

#### 查看所有数据库

```
\l
```
#### 切换数据库

```
\c <databasename> 
```

#### 新建/删除数据库

```
create/drop database <dbname>
```

#### 创建/删除表

```
create table <tbname>()/drop table <tbname>
```

#### 查看当前数据库下所有表

```
\d
```

#### 查看表结构
```
\d <tbname>//相当于desc
```

#### 查看所有相关表格

```
\dt kong_*
```

