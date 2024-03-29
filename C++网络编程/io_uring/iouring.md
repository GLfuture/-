### io_uring 安装

1.kernel在5.10以后（新增三个系统调用，io_uring_setup ,io_uring_register,io_uring_enter）//即ubuntu22.04以后

2.下载liburing 

[2.5.1 与epoll媲美的io_uring.pdf](file:///D:/零声Linux/网络/io_uring/2.5.1 与epoll媲美的io_uring.pdf)

### io_uring使用

[Linux网络编程之异步IO机制io_uring详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/608787533)

io_uring 底层调用mmap，共享内存可以被其他进程访问

### 编译

```shell
gcc main.c -luring -static
```

类似proactor的模式，如：先写再改变user_data的flag为WRITE

### readv:可以读多块不连续的内存

```
man readv
```

### writev（同理）



### 进程间通信:

方法一：iovec  readv   endv

方法二：mmap共享内存映射

```c++
int main()
{
	int sockfd=socket(AF_UNIX,SOCK_DGRAM,0);//创建本地套接字
	if(sockfd==-1) exit(EXIT_FAILURE);
    struct sockaddr_un addr={0};
    addr.sun_family=AF_UNIX;
    strncpy(addr.sun_path,"/tmp/king.sock",sizeof(addr.sun_path)-1);
    int shmemfd=shm_open("",O_RDWR|O_CREAT,S_IRUSR|S_I)
    ftruncate(shmemfd,1024);
    
    void*shmem_ptr;
    shmem_ptr = mmap(NULL,1024,PROT_READ|PROT_WRITE,MAP_SHARED,shmemfd);
        
    struct cmsghdr *cmsg=NULL;
    
    char cmsg_buffer[CMSG_SPACE(sizeof(int))]={0};
    
    struct msghdr msg={0};
    msg.msg_control =msg_buffer;
    msg.msg_controllen=CMG_LEN(sizeof(int));
    msg.msg_name=&addr;
    msg.msg_namelen=sizeof(addr);==
    
    //换个说法就是将cmsg_buffer转换为cmsg
    cmsg = CMSG_FIRSTHDR(&msg);
    cmsg->cmsg_level=SOL_SOCKET;
    cmsg->cmsg_type=SCM_RIGHTS;
    cmsg->cmsg_len=CMSG_LEN(sizeof(int));
    *(int*)CMSG_DATA(cmsg)=shmemfd;
    
    
    sendmsg(sockfd,&msg,0);
}
```



