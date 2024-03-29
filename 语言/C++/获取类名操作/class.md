包含以下两个头文件

```c++
#include<typeinfo.h>
#include<cxxabi.h>
```

包含命名空间

```
using namespace abi;
```

使用宏__cxa_demangle和typeid的name方法

```
std::cout<<__cxa_demangle(typeid(a).name(),NULL,NULL,NULL);
```

