## 重要

[libevent_libev实战那些坑 (1).pdf](file:///D:/零声Linux/c++/libevent/libevent_libev实战那些坑 (1).pdf)

### libevent/libev

libevent/libev是一个事件通知库

libevent中封装了reactor

### reactor 中的io、事件以及他们之间的关系

- io

  （1）io检测

  （2）io操作

  （3）io处理是同步的

- 事件

  （1）读事件

  （2）写事件

  （3）异常事件
  
  （4）事件处理是异步的

#### eventbuffer采用chainbuffer的数据结构

#### 普通数组问题：长度固定，数据挪动很费时

#### ringbuffer问题：长度固定的问题，读取位置的问题

#### chainbuffer的问题：可能会因为一个数据包在不同chainbuffer中导致多次系统调用，但是libevent中通过writev和readv系统接口来进行处理


 ### libevent使用层次

（1）在事件中自己处理IO			基于libevent对事件的处理

（2）libevent内部处理IO			只需要业务逻辑的处理

## libevent的使用

### 事件的类型

```c++
#define EV_TIMEOUT  0x01    //定时事件
#define EV_READ     0x02    //可读事件
#define EV_WRITE    0x04    //可写事件
#define EV_SIGNAL   0x08    //信号事件
#define EV_PERSIST  0x10    //永久事件,EV_PERSIST的作用是：事件被触发后，自动重新对这个event调用event_add函数。
#define EV_ET       0x20    //边缘触发事件，需要I/O复用系统调用支持，比如epoll
```

## event_base结构体

```c++
struct event_base {
    /* eventop 实际上就是对系统提供的I/O复用接口进行封装（select、poll、epoll、kqueue...）,详细介绍见，eventop的描述，这个赋值是在创建event_base的时候 event_base_new-》event_base_new_with_config，在这个函数中从 eventops 结构体数组中选出合适的I/O复用机制。这里使用的是epoll
    */
    const struct eventop *evsel;
    /** 上面选出合适的sIO复用机制之后，通过base->evsel->init生成 evbase 所以可以说evsel 和 evbase 的关系就是c++中类和类的实例的关系*/
    void *evbase;

    /** 使用changelist后，对事件的修改不会立即作用到epoll上，而是把这些修改保存到一个list上，在作用到epoll之前，这些修改可以抵消，合并。比如说，对于同一个事件，先add再del，虽然什么也没做，如果不使用changelist就会多出2个不必要的系统调用。当然如果这种操作并不多的话，使用changelist反而多了额外的开销(相比系统调用，这种开销微不足道)，libevent将选择权交给了你。 */
    struct event_changelist changelist;

    /** 和evsel类似，这里使用的是全局变量 evsigops  */
    const struct eventop *evsigsel;
    /** Data to implement the common signal handler code. */
    struct evsig_info sig;

    int virtual_event_count;/** Number of virtual events */
    int virtual_event_count_max;/** Maximum number of virtual events active */
    int event_count;/** Number of total events added to this event_base */
    int event_count_max;/*Maximum number of total events added to this event_base*/
    int event_count_active;/** Number of total events active in this event_base */
    int event_count_active_max;/*Maximum number of total events active in this event_base */

    /*设置在处理完事件后是否应该终止循环*/
    int event_gotterm;
    /** 设置是否应该立即终止循环 */
    int event_break;
    /** 设置是否应该立即启动一个新的循环实例 */
    int event_continue;

    /** The currently running priority of events */
    int event_running_priority;

    /** Set if we're running the event_base_loop function, to prevent
     * reentrant invocation. */
    int running_loop;

    /** Set to the number of deferred_cbs we've made 'active' in the
     * loop.  This is a hack to prevent starvation; it would be smarter
     * to just use event_config_set_max_dispatch_interval's max_callbacks
     * feature */
    int n_deferreds_queued;

    /* 触发的事件列表，后面会遍历这个列表，执行其中的回调
    把触发的事件插入到列表中的回调流程是event_base_loop-》evsel->dispatch（就是epoll_dispatch）-》evmap_io_active_-》event_active_nolock_-》event_queue_insert_active-》TAILQ_INSERT_TAIL
     */
    struct evcallback_list *activequeues;
    /** The length of the activequeues array */
    int nactivequeues;
    /** A list of event_callbacks that should become active the next time
     * we process events, but not this time. */
    struct evcallback_list active_later_queue;

    /* common timeout logic */

    /** An array of common_timeout_list* for all of the common timeout
     * values we know. */
    struct common_timeout_list **common_timeout_queues;
    /** The number of entries used in common_timeout_queues */
    int n_common_timeouts;
    /** The total size of common_timeout_queues. */
    int n_common_timeouts_allocated;

    /** Mapping from file descriptors to enabled (added) events */
    struct event_io_map io;

    /** Mapping from signal numbers to enabled (added) events. */
    struct event_signal_map sigmap;

    /** Priority queue of events with timeouts. */
    struct min_heap timeheap;

    /** Stored timeval: used to avoid calling gettimeofday/clock_gettime
     * too often. */
    struct timeval tv_cache;

    struct evutil_monotonic_timer monotonic_timer;

    /** Difference between internal time (maybe from clock_gettime) and
     * gettimeofday. */
    struct timeval tv_clock_diff;
    /** Second in which we last updated tv_clock_diff, in monotonic time. */
    time_t last_updated_clock_diff;

#ifndef EVENT__DISABLE_THREAD_SUPPORT
    /* threading support */
    /** The thread currently running the event_loop for this base */
    unsigned long th_owner_id;
    /** A lock to prevent conflicting accesses to this event_base */
    void *th_base_lock;
    /** A condition that gets signalled when we're done processing an
     * event with waiters on it. */
    void *current_event_cond;
    /** Number of threads blocking on current_event_cond. */
    int current_event_waiters;
#endif
    /** The event whose callback is executing right now */
    struct event_callback *current_event;

#ifdef _WIN32
    /** IOCP support structure, if IOCP is enabled. */
    struct event_iocp_port *iocp;
#endif

    /** Flags that this base was configured with */
    enum event_base_config_flag flags;

    struct timeval max_dispatch_time;
    int max_dispatch_callbacks;
    int limit_callbacks_after_prio;

    /* Notify main thread to wake up break, etc. */
    /** True if the base already has a pending notify, and we don't need
     * to add any more. */
    int is_notify_pending;
    /** A socketpair used by some th_notify functions to wake up the main
     * thread. */
    evutil_socket_t th_notify_fd[2];
    /** An event used by some th_notify functions to wake up the main
     * thread. */
    struct event th_notify;
    /** A function used to wake up the main thread from another thread. */
    int (*th_notify_fn)(struct event_base *base);

    /** Saved seed for weak random number generator. Some backends use
     * this to produce fairness among sockets. Protected by th_base_lock. */
    struct evutil_weakrand_state weakrand_seed;

    /** List of event_onces that have not yet fired. */
    LIST_HEAD(once_event_list, event_once) once_events;

};
```

## eventop结构体

```c++
struct eventop {
    /** The name of this backend. */
    const char *name;
    /** 它应该创建一个新结构，其中包含运行后端所需的任何信息，并将其返回。 返回的指针将由 event_init 存储到 event_base.evbase 字段中。 失败时，此函数应返回 NULL */
    void *(*init)(struct event_base *);
    /** 在给定的 fd 或信号上启用读/写。 “事件”将是我们尝试启用的事件：EV_READ、EV_WRITE、EV_SIGNAL 和 EV_ET 中的一个或多个。 'old' 将是之前在此 fd 上启用的那些事件。 'fdinfo' 将是 evmap 与 fd 关联的结构； 它的大小由下面的 fdinfo 字段定义。 第一次添加 fd 时它将设置为 0。 该函数应在成功时返回 0，在错误时返回 -1*/
    int (*add)(struct event_base *, evutil_socket_t fd, short old, short events, void *fdinfo);
    /** As "add", except 'events' contains the events we mean to disable. */
    int (*del)(struct event_base *, evutil_socket_t fd, short old, short events, void *fdinfo);
    /** 事件派遣分发*/
    int (*dispatch)(struct event_base *, struct timeval *);
    /**  释放资源. */
    void (*dealloc)(struct event_base *);
    /** 是否需要重新初始化*/
    int need_reinit;
    /**支持的特性. */
    enum event_method_feature features;
    /** Length of the extra information we should record for each fd that has one or more active events.  This information is recorded as part of the evmap entry for each fd, and passed as an argument to the add and del functions above.*/
    size_t fdinfo_len;
};
```

### 简单创建event_base

```cpp
struct event_base *event_init(void);
struct event_base *event_base_new(void);
void event_base_free(struct event_base *);
```

说明： event_init， event_base_new 函数主要调用event_base_new()函数，返回event_base结构体。

以默认参数创建event_base，从全局变量eventops中选出 操作系统支持的最快方法 。它直接调用event_base_new_with_config。

eventops数组如下所示

```cpp
/* Array of backends in order of preference. */
static const struct eventop *eventops[] = {
#ifdef EVENT__HAVE_EVENT_PORTS
    &evportops,
#endif
#ifdef EVENT__HAVE_WORKING_KQUEUE
    &kqops,
#endif
#ifdef EVENT__HAVE_EPOLL
    &epollops,
#endif
#ifdef EVENT__HAVE_DEVPOLL
    &devpollops,
#endif
#ifdef EVENT__HAVE_POLL
    &pollops,
#endif
#ifdef EVENT__HAVE_SELECT
    &selectops,
#endif
#ifdef _WIN32
    &win32ops,
#endif
    NULL
};
```

### 创建复杂event_base

```cpp
struct event_base *event_base_new_with_config(const struct event_config *);
```

### event_base运行

把事件注册到event_base之后，调用event_base_dispatch就可以等待事件触发，然后执行事件的回调函数了。

```cpp
/*
    阻塞等待事件的到来，如果列表中没有事件，或者事件中调用event_base_loopbreak()或者
event_base_loopexit()。此函数将会退出阻塞。
    event_base_dispatch就是调用了event_base_loop而已，vent_base_loop(event_base, 0);
*/
int event_base_dispatch(struct event_base *);

/*dispatch,在事件触发一次之后就会退出，不管是否还有事件*/
#define EVLOOP_ONCE 0x01
/*设置io为非阻塞*/
#define EVLOOP_NONBLOCK 0x02
/*默认没有事件时，dispatch将推出阻塞，设置这个标志位则没有事件时不会退出阻塞*/
#define EVLOOP_NO_EXIT_ON_EMPTY 0x04
/*flags的枚举见上述所示
正常返回0，
未知错误返回-1，
没有等待或激活的event退出返回1
*/
int event_base_loop(struct event_base *base, int flags);
```

### event_base停止

```cpp
/*让 event_base在给定时间之后停止循环.如果tv参数NULL,event_base会立即停止循环,没有延时.如event_base当前正在执行任何激活事件的回调,则回调会继续运行,直到运行完所有激活事件的回调之才退出.*/
int event_base_loopexit(struct event_base *base,const struct timeval *tv);
/*event_base立即退出循环,不会等待正在执行的事件完成*/
int event_base_loopbreak(struct event_base *base);
//这两个函数都在成功时返回0,失败时返回-1.

/*因为调用 event_base_loopexit() 而退出的时候返回 true*/
int event_base_got_exit(struct event_base *base);
/*因为调用 event_base_break()而退出的时候返回 true*/
int event_base_got_break(struct event_base *base);

/*如果事件处理完后loop会退出，那么在事件的回调中执行此函数则loop不会退出*/
int event_base_loopcontinue(struct event_base *);
```

### event_base的特性获取

```cpp
/*获取Libevent使用的内核I/O复用机制。 */
const char *event_base_get_method(const struct event_base *);

//返回一个字符串数组
const char **event_get_supported_methods(void);

/**
   根据传入的标志，获取event_base中相应事件类型的个数。
   标志的枚举如下所示：
    EVENT_BASE_COUNT_ACTIVE ： count the number of active events, which have been triggered.
    EVENT_BASE_COUNT_VIRTUAL : count the number of virtual events, which is used to represent an internal condition, other than a pending event, that keeps the loop from exiting.
    EVENT_BASE_COUNT_ADDED ： count the number of events which have been added to event base, including internal events.
   @param event_base 
   @param 事件类型
   @return base中相应事件类型的个数
*/
int event_base_get_num_events(struct event_base *, unsigned int);

/**
  根据传入的标志，获取event_base中相应事件类型的个数。和上述的逻辑一样，只是加入了clear控制参数
  @param event_base 
  @param 事件类型
  @param clear 1：重置 相应事件的数量
  @return base中相应事件类型的个数
 */
int event_base_get_max_events(struct event_base *, unsigned int, int);
```

### event_config

event_base的feature的一些参数设置

```cpp
struct event_config {
    TAILQ_HEAD(event_configq, event_config_entry) entries;

    int n_cpus_hint;
    struct timeval max_dispatch_interval;
    int max_dispatch_callbacks;
    int limit_callbacks_after_prio;
    enum event_method_feature require_features;
    enum event_base_config_flag flags;
};

/**
   设置特性时，不能直接对event_config进行赋值，因为它对外是不透明的。可以使用event_config_**系列函数设置相应的特性。
*/
struct event_config *event_config_new(void);
void event_config_free(struct event_config *cfg);

/*  设置是base不使用指定的I/O复用机制。
    method可选值见 eventops。
*/
int event_config_avoid_method(struct event_config *cfg, const char *method);
```

###  event_base的feature

```cpp
/*用来描述event_base(必须)提供哪些特性的标志。*/
enum event_method_feature {
    /** 要求支持边沿触发的后端 */
    EV_FEATURE_ET = 0x01,
    /*要求添加、删除单个事件,或者确定哪个事件激活的操作是O(1)复杂度的后端. */
    EV_FEATURE_O1 = 0x02,
    /** 要求支持任意文件描述符,而不仅仅是套接字的后端. */
    EV_FEATURE_FDS = 0x04,
    /** 要求可以使用EV_CLOSED来检测连接关闭，而不需要读取所有挂起的数据。支持EV_CLOSED的方法
    可能不能在所有内核版本上提供支持。*/
    EV_FEATURE_EARLY_CLOSE = 0x08
};

/*  由于操作系统的限制，并不是每一个I/O复用机制都支持所有的特性。可以使用此函数设置在创建base时只
    选择支持指定特性的I/O复用机制。
    feature的枚举值见 event_method_feature
*/
int event_config_require_features(struct event_config *cfg, int feature);

/*获取base支持的特性
*/
int event_base_get_features(const struct event_base *base);
```

### event_base的flag

```cpp
enum event_base_config_flag {
    /** 不要为 event_base分配锁.设置这个选项可以为event_base节省一点用于锁定和解锁的时
    间,但是让在多个线程中访问 event_base成为不安全的.*/
    EVENT_BASE_FLAG_NOLOCK = 0x01,
    /** 选择使用的后端时,不要检测EVENT_*环境变量 */
    EVENT_BASE_FLAG_IGNORE_ENV = 0x02,
    /** 仅用于 Windows,让 libevent在启动时就启用任何必需的IOCP分发逻辑,而不是按需启用.*/
    EVENT_BASE_FLAG_STARTUP_IOCP = 0x04,
    /** 不是在每次事件循环准备好运行超时回调时检查当前时间，而是在每次超时回调之后检查。*/
    EVENT_BASE_FLAG_NO_CACHE_TIME = 0x08,

    /** 告诉 base,如果决定使用epoll后端,可以安全地使用更快的基于 changelist的后端.epoll-changelist后端可以在后端的分发函数调用之间,同样的fd多次修改其状态的情况下,避免不必要的系统调用.但是如果传递任何使用 dup ()或者其变体克隆的 fd给libevent, epoll-changelist后端会触发一个内核bug,导致不正确的结果.在不使用epoll后端的情况下,这个标志是没有效果的.也可以通过设置 EVENT_EPOLL_USE_CHANGELIST:环境变量来打开epoll-changelist选项.*/
    EVENT_BASE_FLAG_EPOLL_USE_CHANGELIST = 0x10,

    /** 通常，Libevent使用我们所拥有的最快的单调计时器来实现它的时间和超时代码。但是，如果设置了这个标志，我们将使用效率较低的更精确的计时器(假设存在计时器)。*/
    EVENT_BASE_FLAG_PRECISE_TIMER = 0x20
};

/* 通过flag设置base的某些参数特征，flag的枚举值见event_base_config_flag*/
int event_config_set_flag(struct event_config *cfg, int flag);
```

## event结构体（一般在libevent第一种使用层次时建立该对象）
```c++
struct event {
    TAILQ_ENTRY (event) (ev_active_next);
    TAILQ_ENTRY (event) (ev_next);//
    int min_heap_idx;   /* for managing timeouts */

    struct event_base *ev_base;
    
    evutil_socket_t ev_fd;
    union {
        /* used for io events */
        struct {
            TAILQ_ENTRY (event) (ev_io_next);
            struct timeval ev_timeout;
        } ev_io;
    
        /* used by signal events */
        struct {
            TAILQ_ENTRY (event) (ev_signal_next);
            short ev_ncalls;
            /* Allows deletes in callback */
            short *ev_pncalls;
        } ev_signal;
    } _ev;
    
    short ev_events;//是event关注的事件类型，它主要包括输入输出时间，定时时间和信号
    
    struct timeval ev_timeout;
    
    int ev_pri;     /* smaller numbers are higher priority */
    
    /* allows us to adopt for different types of events */
    void (*ev_closure)(struct event_base *, struct event *);
    void (*ev_callback)(evutil_socket_t, short, void *arg);
    void *ev_arg;
    
    int ev_res;     /* result passed to event callback */
    int ev_flags;
};
```

## eventbuffer结构体（通常在libevent第二个使用层次中使用）

```
struct event_watermark wm_read;
struct event_watermark wm_write;
```

低水平触发：buffer中有多少数据就要触发回调

高水平触发：buffer数据较多时不再处理事件

```
struct evbuffer *input;//读缓冲区
struct evbuffer *output;//写缓冲区
bufferevent_data_cb readcb;//读事件回调
bufferevent_data_cb writecb;//低水平触发回调
bufferevent_data_cb errorcb//被动关闭连接或其他异常回调
```

## evconnlistener结构体

```c++
//一系列的工作函数，因为listener可以用于不同的协议。
struct evconnlistener_ops {
	int (*enable)(struct evconnlistener *);
	int (*disable)(struct evconnlistener *);
	void (*destroy)(struct evconnlistener *);
	void (*shutdown)(struct evconnlistener *);
	evutil_socket_t (*getfd)(struct evconnlistener *);
	struct event_base *(*getbase)(struct evconnlistener *);
};

//一层一层封装，加上隔离
struct evconnlistener {
	const struct evconnlistener_ops *ops;	//操作函数
	void *lock;								//锁变量，用于线程安全
	evconnlistener_cb cb;					//用户的回调函数
	evconnlistener_errorcb errorcb;			//发生错误时的回调函数
	void *user_data;               			//回调函数的参数，当回调函数执行时候，通过形参传入回调函数内部
	unsigned flags;                			//属性标志 ，例如socket套接字属性，可以是阻塞，非阻塞，reuse等。
	short refcnt;                  			//引用计数
	unsigned enabled : 1;					//位域为1.即只需一个比特位来存储这个成员 
};
struct evconnlistener_event {
	struct evconnlistener base;
	struct event listener;     //内部event,插入到event_base，完成监听
};

```

## libevent模板

```cpp
#include<event.h>
#include<sys/epoll.h>
#include<event2/listener.h>
#include<string.h>
#include<stdlib.h>
#include<sys/socket.h>
#include<netinet/in.h>
#include<arpa/inet.h>
#include<iostream>
#include<errno.h>
//struct bufferevent *bev, short what, void *ctx
//连接redis的回调函数
//void conneted_cb(struct bufferevent* bev, short what, void* ctx)
//{
//	if (what == BEV_EVENT_CONNECTED){
//		std::cout << "redis  connect  successfully!" << std::endl;
//	}
//	else
//	{
//		std::cout << "redis  connect  failed!" << std::endl;
//	}
//}

//读回调
void  read_cb(struct bufferevent* bev, void* ctx)
{
	struct evbuffer* evbuf = bufferevent_get_input(bev);
	char* msg = evbuffer_readln(evbuf, NULL, EVBUFFER_EOL_LF);
	std::cout << msg << std::endl;
	if (!msg) return;
	std::string str = "";
	str = msg;
	delete msg;
	bufferevent_write(bev, str.c_str(), str.length());

}

//其他异常情况回调
void error_cb(struct bufferevent* bev, short what, void* ctx)
{
	if (what & BEV_EVENT_EOF) //read=0
		std::cout << "connection closed" << std::endl;
	else if(what & BEV_EVENT_ERROR)//strerro(errno)
		std::cout << strerror(errno) << std::endl;
	else if(what & BEV_EVENT_TIMEOUT)//超时
		std::cout << "timeout" << std::endl;
	bufferevent_free(bev);//close(fd)
}

void accept_cb(struct evconnlistener* listen, evutil_socket_t fd, struct sockaddr* sock, int socklen, void* arg)
{
	char ip[32];
	memset(ip, 0, sizeof(ip));
	evutil_inet_ntop(AF_INET, sock, ip, sizeof(ip));
	std::cout << "fd:" << fd << "  " << "ip:" << ip << std::endl;
	struct event_base* base = (struct event_base*)arg;
	//设置回调
	struct bufferevent* bev = bufferevent_socket_new(base, fd, BEV_OPT_CLOSE_ON_FREE);
	bufferevent_setcb(bev, read_cb, NULL, error_cb, NULL);
	bufferevent_enable(bev, EV_READ | EV_PERSIST);//注册读事件和持久事件
	//evutil_closesocket(fd);//关闭连接
	//连接redis
	/*sockaddr_in sin = {0};
	sin.sin_addr.s_addr = inet_addr("127.0.0.1");
	sin.sin_port = htons(6379);
	sin.sin_family = AF_INET;
	struct bufferevent* ev = bufferevent_socket_new(base, -1, BEV_OPT_CLOSE_ON_FREE);
	bufferevent_socket_connect(ev, (struct sockaddr*)&sin, sizeof(sin));
	bufferevent_setcb(ev, NULL, NULL, conneted_cb,NULL);*/
}

int main()
{
	struct event_base* base = event_base_new();

	struct sockaddr_in sin = { 0 };
	sin.sin_family = AF_INET;
	sin.sin_addr.s_addr = htonl(INADDR_ANY);
	sin.sin_port = htons(10000);
	struct evconnlistener* listen = evconnlistener_new_bind(base,accept_cb,base,LEV_OPT_REUSEABLE|LEV_OPT_CLOSE_ON_FREE,10,(struct sockaddr*)&sin,sizeof(sin) );
	event_base_dispatch( base );
	evconnlistener_free(listen);
	event_base_free(base);
	return 0;
}
```

