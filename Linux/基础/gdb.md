## gdb调试

g++编译时加入-g

而后执行gdb ./demo.exe

其中b（添加断点）+行号，r（运行），c（继续执行），q（退出）,n(下一步)，p(打印变量)

info thread 显示线程

thread id 进入对应线程

handle SIGINT nostop print pass  打印信号处理函数