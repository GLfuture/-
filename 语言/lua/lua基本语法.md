https://www.runoob.com/manual/lua53doc/contents.html

### 支持多返回值

只有当函数调用在最后一个时，返回多个值，否则返回一个

```
function func() return a,b;
local x,y,z = func(),30;//x=a,y=b,z=30
local x,y,z=30,func();//x=30,y=a,z=nil
```

### 可变参数

```
local tb = table.pack(...)//打包可变参数，内部封装n为参数个数，吧可变参数转换为一个列表
print(table.unpack(tb))//
```

 ### 元方法

```
__index=function(table,key)//当table不是表或者表中没有key会调用该函数
__newindex
```

### 语法糖

```
ud:echo
等价于
ud.echo(ud)
```

