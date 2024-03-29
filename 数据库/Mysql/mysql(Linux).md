 要调用mysql.h，必须安装mysql-client

修改配置文件/etc/mysql/mysql.conf.d/mysqld.cnf

引入头文件路径 -I

```shell
g++ 1.cpp -o 1.exe -I /usr/include/mysql/
```

引入库 -l

```shell
g++ 1.cpp -o 1.exe -I /usr/include/mysql/ -lmysql.client
```

引入库路径 -L

安装开发包

```
sudo apt-get install libmysqld-dev

sudo apt-get install libmysqlclient-dev
```

初始化密码：

```
sudo mysql -u root -p此时不需要密码
登录后使用
alter user 'root'@'localhost' identified with mysql_native_password by '密码';改密码
```

