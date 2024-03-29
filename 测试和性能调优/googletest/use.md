

```shell
//如果测试单元没有main函数，就引入gtest_main静态库
g++ test.cc -o test -lgtest -lgtest_main -lpthread
//使用mock打桩
g++ test.cc -o test -lgtest -lgmock -lpthread 
```

测试内容：

​	1.功能正确

​	2.边界情况

​	3.异常情况

测试套件使用基本步骤：

```
 TEST(name1,name2)//name1相同的测试案例属于同一个测试套件
```

---

在多个测试套件需要同一组测试数据时，需要用到测试夹具

测试夹具使用基本步骤：

```
class Sample : public Testing::Test //先继承Test

TEST_F(fixture,name2)//fixture必须和继承类名相同
```

### 测试逻辑

```
EXPECT_*	/同一测试案例下出错继续执行
ASSERT_*	//同一测试案例下出错不执行
```

注：返回均为流对象，可以EXPECT_*()<<"this is error";

### 死亡测试

```
EXPECT_DEATH(function(),stdout) //参数：方法调用，错误信息，
```

注：死亡测试不管写在哪个位置，都会出现在该测试案例的最前面

---

如果测试逻辑判断比较复杂

SUCCEED()

FAIL()

```
if(condition){
	SUCCEED()
}else{
	FAIL()
}
```

### 谓词断言(错误信息比EXCEPT_TRUE多一些,会把参数值打印出来)

``` 
EXPECT_PREDn(func,value...) //n改为对应func的参数个数
```

### 类型参数化

```
using testing::Test;
using testing::Types;
//先声明测试夹具
template<class T>
class TestFixtureSmpl : public testing::Test {
protected:
	void SetUp();//测试夹具测试前调用的函数			--做初始化的工作
	void TearDown();//测试夹具测试后调用的函数 		--做清理的工作
};
//枚举测试类型
typedef Types<class1,class2,class3,...> Implementations;
//#define TYPED_TEST_SUITE(CaseName,Types,__VA_ARGS__...)
//casename 一定要与测试夹具名字一致
TYPED_TEST_SUITE(TestFixtureSmpl,Implementations);
//#define TYPED_TEST(casename,Testname)
//开始测试,casename同样是测试夹具
TYPED_TEST(TestFixtureSmpl,Testname)

```

### 事件

```
在TestEventListener中有很多埋点，在每个测试案例、套件、单元测试开始和结束都埋下点位
```

步骤：

```
1.继承EmptyTestEventListener
2.实现对应事件点位 
```

### mock

```
1.约定一个基类中有需要测试的接口
2.编写一个mock类继承基类
3.在mock类中，MOCK_METHODn(func,void(int distance ..)),n为参数个数
```

[C++开发测试工具gmock使用详解（进阶）——对抽象接口类进行gmock打桩并测试_gmock stub例子_wendy_ya的博客-CSDN博客](https://blog.csdn.net/didi_ya/article/details/123275719)