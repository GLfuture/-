

### 初识

moudle: ko文件

通过 insmod kernel_module.ko 加入内核模块

通过rmmod kernel_module.ko 移除模块

module用 .c编写,通过Makefile编译

### 实现函数

```c++
module_param //模块参数传入

module_init	//模块进入入口函数

module_exit//模块退出入口函数
```

### 插入/删除模块流程

1.通过makefile编译代码

```makefile
obj-m := program2.o

KERNELBUILD := /lib/modules/$(shell uname -r)/build

default:
	make -C $(KERNELBUILD) M=$(shell pwd) modules

clean:
	rm -rf *.o *.ko *.mod.c .*.cmd *.markers *.order *.symvers .temp_versions
```

2.通过insmod将模块插入内核

```shell 
sudo insmod *.ko
```

3.通过dmesg查看内核信息

```shell
dmesg
dmseg -C //清空内核模块信息
```

4.通过rmmod移除模块

```shell
rmmod
```

### 查看模块

```
lsmod
```

### module带参

```c++
module_param(debug,bool,S_IRUSR);
```

运行命令

```shell
sudo insmod program2.ko debug=1
```



### 常用函数

```c++
int printk()//内核的printf
pid* find_get_pid(pid_t);
task_struct* get_pid_task(pid*)//根据pid拿到对应的task_struct
for_each_process(task_struct* task)//遍历task

```

demo

```
#include <linux/module.h>
#include <linux/pid.h>
#include <linux/sched.h>
#include <linux/sched/signal.h>
#include <linux/init.h>
#include <linux/fs_struct.h>
#include <linux/fdtable.h>


bool debug = 0;
module_param(debug, bool, S_IRUSR);

pid_t pid = 0;
module_param(pid, int, S_IRUSR);

static int kernel_module_init(void)
{
    struct pid *spid;
    struct task_struct *task;

    printk("module init\n");
    if (pid <= 0) return -1;
    spid = find_get_pid(pid);
    task = get_pid_task(spid, PIDTYPE_PID);
    if(!task) return -1;
    printk("name : %s\n",task->comm);

    for_each_process(task){
        printk("name : %s[%d]\n",task->comm,task->pid);
    }

    return 0;
}
static void kernel_module_exit(void)
{
    printk("exit\n");
}

module_init(kernel_module_init);

module_exit(kernel_module_exit);

MODULE_LICENSE("GPL");
```



