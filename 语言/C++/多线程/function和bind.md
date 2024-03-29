###      function

类似函数指针

```c++
void func1(promise<string>& pro)
{
    pro.set_value("Hello");
}
int main()
{
    function<void(promise<string>&)> fn=&func1;//&可要可不要
    promise<string> sp;
    future<string> future = sp.get_future();
    fn(std::ref(sp));
    cout << future.get() << endl;
    return 0;
}
```

### bind（绑定类的成员函数的时候也要绑定对象的地址）

```c++
1.只是绑定函数，参数自己手动传入（使用std占位参数）std::place holder::_1
2.绑定函数的时候也把参数绑定
bind(&函数名)
```

绑定重载函数

```c++
class demo {
public:
    void func1(int a)
    {
        cout << a << endl;
    }
	void func1()
	{
	}
};

int main()
{
    demo m;
    auto f4 = bind((void(demo::*)())&demo::func1, &m);//必须指明函数
    auto f5 = bind((void(demo::*)(int))&demo::func1, &m, 1)
    return 0;
}

```



