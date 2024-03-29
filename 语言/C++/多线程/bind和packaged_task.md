bind和packaged_bind连用

```c++
template<class T,class Argv>
void func(T&& f,Argv argv)
{
    using In = decltype(f(argv,2));
    auto task = make_shared<packaged_task<In(int)>>(bind(forward<T>(f), forward<Argv>(argv),std::placeholders::_1));//绑定时不确定绑定的参数时可以使用占位参数,packaged_task的模板对应返回后的返回值和参数类型，如果全部参数绑定即不需要输入，就用packaged_task<In()>
    (*task)(9);
}
void temp(int a,int b)
{
    cout << a << b <<endl;
}
int main()
{
    func(temp,10);
    return 0;
}
```

