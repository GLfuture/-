### 将是skynet移至项目目录

### config编写

```
thread=8//底部线程数
logger=nil//日志存放位置,nil打印在控制台，可以写文件名
logpath="."//log文件保存路径
harbor=0//不使用集群,使用为1
start="main"//第一次启动的服务
lua_path="./skynet/lualib/?.lua;./skynet/lualib/init.lua"
-- lua抽象的进程
luaservice="./skynet/service/?.lua;./app/?.lua;"
lualoader="./skynet/lualib/loader.lua"
cpath="./skynet/cservice/?.so"
lua_cpath="./skynet/luaclib/?.so"
```

### Makefile编写

```
SKYNET_PATH ?= ./skynet
all :
	cd $(SKYNET_PATH) && $(MAKE) PLAT='linux'
clean :
	cd $(SKYNET_PATH) && $(MAKE) clean
```



