### etcd

C++ client环境搭建

1.boost库依赖

[Boost C++ Libraries - Browse /boost at SourceForge.net](https://sourceforge.net/projects/boost/files/boost/)

```
./boot_trap
```

2.安装grpc和protobuf

3.安装cpprestsdk

```
 git clone https://github.com/microsoft/cpprestsdk.git
```

需要修改Release/CMakeLists.txt

```
set(WERROR ON CACHE BOOL "Treat Warnings as Errors.")
改为
set(WERROR OFF CACHE BOOL "Treat Warnings as Errors.")
```

4.安装etcd-cppapi-v3

```
https://github.com/etcd-cpp-apiv3/etcd-cpp-apiv3.git
```

需要修改CMakeLists

```
 git clone https://github.com/microsoft/cpprestsdk.git
```

 [ETCD.pdf](D:\零声Linux\etcd\ETCD.pdf) 

### etcd启动

配置文件启动

```yaml
# 节点名称
name: "etcdnode0"
#数据存储目录（容器内）
data-dir: "/etcd-data/data"
#预写时日志
wal-dir: "/etcd-data/wal"
#集群成员之间通讯使用的URL
listen-peer-urls: "http://0.0.0.0:2380"
#集群提供给外部客户端访问的url
listen-client-urls: "http://0.0.0.0:2379"


#集群配置
initial-advertise-peer-urls: "http://192.168.150.132:2380"
#集群初始成员配置，是etcd静态部署的核心初始化配置，它说明了当前集群由哪些URLs组成
initial-cluster: "etcdnode0=http://192.168.150.132:2380,etcdnode1=http://192.168.150.132:12380,etcdnode2=http://192.168.150.133:2380"
#初始化集群状态，如果集群存在就为existing ，新建则为new
initial-cluster-state: "new"
#引导期间etcd集群的初始集群令牌，防止不同集群产生交互
initial-cluster-token: "etcd-cluster"
# 向客户端发布的服务点
advertise-client-urls: "http://192.168.150.132:2379"
logger: "zap"
#配置日志级别(debug,info,warn,error,panic,fatal)
log-level: "warn"
log-outputs: 
  - "stderr"
```

### docker 启动

```
docker run -d -p 2379:2379 -p 2380:2380 -v /tmp/etcd0-data:/etcd-data -v /home/gong/work/etcdconf:/etcd-conf --name etcd0 quay.io/coreos/etcd:v3.5.7 etcd --config-file=/etcd-conf/etcd0.yaml
```

### etcdctl

```
endpoint health/status		//默认2379，节点客户端端口
--endpoints=http://ip:port		//指定地址
-w table/json/fields			//显示模式
--cluster 						//为集群
--prefix ""						//与get连用，查找前缀为""
--prev-kv ""					//byte比较，返回小于等于prev-kv的key-value

lease grant	[seconds]			//返回id
put key1 value --lease [id]		//租约key1
--ignore-lease					//在put时使用，如果没有这一行，put时会取消租约
lease keep-alive				//刷新租约（服务注册和发现会用到）
lease list						//显示所有租约
lease revoke					//撤销租约
lease timetolive				//查看租约剩余时长
```

##### 锁

```
etcdctl lock mymutex			//创建锁,如果没有人持有锁则响应，有则阻塞,当锁被释放，下一个就立刻持有锁(检测不到版本号小的了就持有锁)
etcdctl get "mymutex" --prefix	//查看锁
```

##### 选举

```
etcdctl elect mymutex node1
etcdctl get "mymutex" --prefix	//比锁多出value值（为node1）
```

##### 事务

```
txn 								//在一个事务中处理所有请求
	-i,--interactive[=false]		//在交互模式下输入事务会分成条件，成功执行，失败执行三部分
```

 

