1.结果集为释放

2.当delete多条数据的时候，会提示不是安全模式，删除的条件中必须有主键或者将sql设置成安全模式（在workbench中会有安全警报，但在navicat中会将所有满足条件的记录全部删除）

解决方案(创建存储过程，关闭安全模式后再打开):

```mysql
SET SQL_SAFE_UPDATES=0;
DELETE ....
SET SQL_SAFE_UPDATES=1;
```

3.数据库关闭

4.查询结果中带有NULL会导致程序崩溃

5.mysql.conf文件中设置为本地回环导致远程连接不上

