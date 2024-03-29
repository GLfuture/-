作者：程序员Derozan
链接：https://www.zhihu.com/question/47845878/answer/2913995917
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



在程序开发中，经常需要通过执行`命令行`操作来拿到一些系统信息，比如获取`进程信息`，获取系统所有`用户`等等，在这种情况下我们不但需要执行命令行，还需要拿到命令行的`返回结果`。`C++`提供了一个`system`函数，可以执行指定的`cmd`，但是只能返回一个执行结束的`状态`，不能获得执行后的结果，所以需要我们自己来造轮子。

## **执行操作** 

 除了[system函数](https://www.zhihu.com/search?q=system函数&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2913995917})以外，我们还可以通过`popen`这个函数，这个函数的含义是

```text
#include <stdio.h>

FILE *popen(const char *command, const char *type);

int pclose(FILE *stream);
```

>  popen()函数的作用是:通过创建管道、fork、并调用shell。因为根据定义，管道是单向的，类型参数可以只指定读或写作，不是两者兼而有之;结果流相应地被读取只写或只写。
>
>  

 用法很简单，准备好cmd，对于`type`，由于我们是读取command的`输出`而不是向command输入参数，所以设置为`"r"`,然后popen会返回一个`文件结构体指针`，用于`读取`执行完命令的`输出`，如果type是`"w"`的话，则这个文件[结构体指针](https://www.zhihu.com/search?q=结构体指针&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2913995917})用于向cmd输入。

 执行cmd的逻辑为

```text
#include <stdio.h>

int execCmd(const char* cmd, std::string& result){
    FILE* pipe = popen(cmd, "r");
    if(!pipe){
        printf("popen error\n");
        return -1;
    }
    return 0;
}
```

 接下来就是`读取`cmd的输出

## **读取操作** 

 读取[pipe](https://www.zhihu.com/search?q=pipe&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2913995917})的内容有多种方式，可以通过`getline`，`fgets`以及`fread`来读取

### **fgets读取**

```text
int execCmdUseFgets(const char *cmd, std::string &result)
{
    FILE *pipe = popen(cmd, "r");
    if (!pipe)
    {
        printf("popen error\n");
        return -1;
    }
    size_t ret = 0;
    char buf[bufferLen + 1] = {0};
    while (fgets(buf, bufferLen, pipe) != NULL)
    {
        result.append(buf);
    }
    printf("buf=%s\n",result.c_str());
    return 0;
}
```

### **getline读取**

```text
int execCmdUseGetline(const char *cmd, std::string &result)
{
    FILE *pipe = popen(cmd, "r");
    if (!pipe)
    {
        printf("popen error\n");
        return -1;
    }
    size_t ret = 0;
    char* buf = nullptr;
    size_t len  = bufferLen;
    ssize_t ret = 0;
    while ((ret = getline(&buf, &len, pipe)) != -1)
    {
        result.append(buf);
    }
    
    printf("buf=%s\n", result.c_str());
    return 0;
}
```

### **fread读取**

```text
int execCmd(const char* cmd, std::string& result){
    FILE* pipe = popen(cmd, "r");
    if(!pipe){
        TdError("popen error");
        return -1;
    }
    size_t ret = 0;
    char buf[bufferLen+1] = {0};
    while ((ret = fread(buf, sizeof(char),bufferLen, pipe)) == bufferLen){
        result.append(buf);
    }
    if(ferror(pipe) != 0) {
        TdError("read pipe error");
        return -1;
    }
    if(feof(pipe) != 0) {
        TdError("exec cmd:%s success");
        result.append(buf, ret);
    }
    return 0;
}
```