### example:

```c++
bpftrace -e 'uprobe:/usr/lcoal/nginx/sbin/nginx:ngx_close_connection /cond/ {printf("close connection\n")}'//对应应用程序的哪个函数 +/条件/ + {执行动作}

bpftrace -l '*accept*'  //查找内核中所有通配*accept*的挂载点
```

```c++
//过滤除nginx进程之外的数据
tracepoint:syscalls:sys_enter_read
/  comm == "nginx"  /
{
	printf("read  %s  %d\n",comm,pid);
}
```

```c++
tracepoint:syscalls:sys_enter_write
/  comm == "nginx"  /
{
	printf("write  %s  %d  ,buf:%s\n",comm,pid,str(args->buf));
}
```

##### 实现类似于tcpdump的功能

```
#include <net/sock.h>
BEGIN
{
	printf("%8s  %6s %15s","TIME","PIS","COMM");
	printf("%20s %6s %20s %6s\n","RADDR","RPORT","LADDR","LPORT");
}

kretprobe:inet_csk_accept{
	
	$sk = (struct sock*) retval; //retval：bpf内置
	$raddr = ntop($sk->__sk_common.skc_daddr);
	$laddr = ntop($sk->__sk_common.skc_rcv_saddr);
	
	//bpf内置
	time("%H:%M:%S");
	printf("%6d %15s",pid,common);
	printf("%20s,%20s",$raddr,$laddr);
}


```

##### 读取文件io

```
#include <linux/fs.h>
#include <linux/path.h>
#include <linux/dcache.h>

kprobe:vfs_open 
/comm == "cat"/
{
	printf("vfs_open: %s ,name : %s",comm,str(((struct path*)arg0)->dentry->d_name.name));
}

kprobe:vfs_write
/comm == "cat"/
{
	$file = str(((struct file*)arg0)->fpath.dentry->d_name.name);
	printf("vfs_write: %s, count: %d, buf:%s \n",$file , arg2 , str(arg1));
}
```

##### nvme裸盘

```
#include <linux/nvme_ioctl.h>

kprobe:nvme_submit_io
{
	printf("%s",str(((struct nvme_user_io*)arg1)->addr)); 
}
```



### 参数

```
-e   //执行
-l   //列出
-v	 //查看参数，与-l连用
```

[7.2.1 内核bpf的实现原理.pdf](file:///D:/零声Linux/Linux/内核/7.2.1 内核bpf的实现原理.pdf) 

### 脚本执行

bpftrace *.bt

*.bt  :

```
uprobe:/usr/lcoal/nginx/sbin/nginx:ngx_close_connection  {
	printf("close connection\n")
};
```

### 常用tracepoint

syscalls//系统调用

kprobe//内核函数

uprobe//用户态函数

```c++
//网络
tracepoint:syscalls:sys_enter_accept
tracepoint:syscalls:sys_enter_accept4
tracepoint:syscalls:sys_enter_read
tracepoint:syscalls:sys_enter_connect
tracepoint:syscalls:sys_enter_write	
kprobe:inet_csk_accept //accept
//磁盘IO
tracepoint:syscalls:sys_*
kprobe:vfs_open 
kprobe:nvme_sumit_io 
```

 [7.2.2 bpf对内核功能的观测.pdf](..\..\..\..\零声Linux\Linux\内核\7.2.2 bpf对内核功能的观测.pdf) 