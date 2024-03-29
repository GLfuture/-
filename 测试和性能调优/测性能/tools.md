### 测网络性能（网络状况）

测试网络吞吐量

tool:			 iperf/iperf3

```
iperf3 参数
-s //server启动
-c ip //client端需要连接的server的ip地址
-u   //udp包，默认tcp
-b  200m //测udp时会用到该参数,默认udp 1mb/sec,tcp 无上限
```

可能导致网络传输性能差异的要素

```
1.cwnd大小的差异 //拥塞窗口设置大小
```

监测网卡数据

tool:				tcpdump

```
tcpdump -i eth0 udp port 10000 
```

### 测磁盘性能和文件系统io性能

tool：			fio

测磁盘性能:    filename = /dev/sda    //磁盘文件

测文件系统性能:	filename = /home/...	//文件路径

当测试文件系统的io时，必须指定 --size

```
./fio --name=king  --ioengine=psync(io_uring...) --iodepth=1 --rw=randread(randwrite) --bs=4k --numjobs=1(线程数) --size=1G --runtime=20 --filename=./fiotestfile
```

```
参数
-size //数据大小
-name //指定名字
-ioengine //psync/libaio/io_uring/spdk_nvme/bdev
-filename //文件路径或者磁盘设备文件
-iodepth  //异步io请求存储的环形队列的大小              
-numjobs //执行线程数
```

### 测试mysql磁盘IO

tool:				mysqlslap

```
-h 
-p //password
--concurrency=100：指定同时有 100 个客户端连接
--number-of-queries=1000：指定总的测试查询次数（并发客户端数 * 每个客户端的查询次数），这样本样例平均每个客户端查询 10 次
--iterations //测试的次数
--query //测试的sql语句
--only-print //不连接数据库。只打印它会做的事情
-a --auto-generate-sql//自动生成测试案例
```

demo:

```
mysqlslap -q "insert into PA.teacher(name) values ('gong');" -uroot -p --concurrency=100 --number-of-queries=1000
```

[mysqlslap介绍和使用-CSDN博客](https://blog.csdn.net/red98/article/details/103513468)

### 测redis性能

tool:					redis-benchmark

### 测http服务器性能

tool:					wrk

```
-c, --connections: total number of HTTP connections to keep open with
                   each thread handling N = connections/threads //连接数

-d, --duration:    duration of the test, e.g. 2s, 2m, 2h	//测试时间

-t, --threads:     total number of threads to use

-s, --script:      LuaJIT script, see SCRIPTING

-H, --header:      HTTP header to add to request, e.g. "User-Agent: wrk"

    --latency:     print detailed latency statistics

    --timeout:     record a timeout if a response is not received within
                   this amount of time.

```

