atomic

[(54条消息) C++多线程——原子操作atomic_princeteng的博客-CSDN博客](https://blog.csdn.net/princeteng/article/details/103934620)

[原子操作 CAS 与锁实现.pdf](file:///D:/零声Linux/c++/多线程/原子操作 CAS 与锁实现.pdf)

原子变量解决了：

​	1.原子执行的问题

​	2.可见性的问题（load，store）

​			store写操作可以让其他线程对修改的数据可见

​			load读操作可以看见其他线程最新操作的数据

​	3.执行序的问题（在std中），一般用作内存栅栏和load、store的参数


自己的代码:

```c++
#include<iostream>
#include<thread>
#include<mutex>
#include<atomic>
#if defined(_WIN32)
#include<windows.h>
#define sleep Sleep
#define usleep Sleep
#elif defined(__linux__)
#include<unistd.h>
#endif
#define THREAD_LENGTH 10
using namespace std;
void Count(atomic<int> * count)
{
    for (int i = 0; i < 10000; i++)
    {
        (*count)++;
        usleep(1);
    }
}
int main()
{
    thread** PList = new thread * [10];
    atomic<int> count(0);
    for (int i = 0; i < THREAD_LENGTH; i++)
    {
        PList[i] = new thread(Count, &count);
    }
    for (int i = 0; i < 100; i++)
    {
        cout << "count :" << count << endl;
        sleep(1);
    }
    return 0;
}
```



##  CAS(无锁队列)

[(54条消息) c++11总结23——CAS（无锁队列）_却道天凉_好个秋的博客-CSDN博客_c++ cas](https://blog.csdn.net/www_dong/article/details/119920236)

[(54条消息) C++11 atomic及其内存序列和CAS_c++ atomic_shldy1999的博客-CSDN博客](https://blog.csdn.net/qq_44875284/article/details/123994575?ops_request_misc=%7B%22request%5Fid%22%3A%22167698849916800184178024%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=167698849916800184178024&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-6-123994575-null-null.142^v73^control_1,201^v4^add_ask,239^v2^insert_chatgpt&utm_term=cas  c%2B%2B&spm=1018.2226.3001.4187)