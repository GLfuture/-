## condition_variable

条件变量

wait（unique_lock<mutex>  mutex）//无条件等待唤醒

wait（unique_lock<mutex>  mutex,[](){ }）//条件等待，条件返回值只能是bool，返回false阻塞线程

notify_one()//唤醒一个线程

notify_all()//唤醒全部进程



附：

---

unique_lock解释

[(55条消息) std::unique_lock 介绍_小米的修行之路的博客-CSDN博客_unique_lock](https://blog.csdn.net/u012372584/article/details/96852295)

[(55条消息) 【学习笔记】C++并发与多线程笔记五：unique_lock详解_c++ unique_lock_天上下橙雨的博客-CSDN博客](https://blog.csdn.net/weixin_40026797/article/details/125380206)