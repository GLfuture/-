windows要包含winsock2.h头文件,预编译#pragma comment(lib,"ws2_32.lib")

linux包含sys/socket.h，sys/type.h，unistd.h，netinet/in.h

和TCP客户端的流程一样：

1.创建socket

2.初始化 sockaddr_in（AF_INET协议族，目的IP，目的端口）

3.connect连接主机（可有可无，但是在UDP第一次发送时很容易丢失，connect相当于为sendto开辟道路）

4.sendto发送UDP报文

5.recvfrom接收请求报文

UDP具备，tcp不具备的好处：

1.UDP传输速度快，没有带宽限制（下载）

2.UDP响应速度快（游戏）

3.UDP能实现广播和组播

4.UDP能实现p to p,而TCP只能连接一个公网ip或者局域网内的ip，不能实现两个局域网之间主机的通信,而UDP可以通过打洞实现两个局域网之间主机的通信

（DNS）关于UDP，系统提供了接口函数 gethostbyname(char * hostname) 返回一个hostent类型，失败返回NULL

struct hostent {
      char  *h_name;            /* official name of host */
      char **h_aliases;         /* alias list */
      int    h_addrtype;        /* host address type */
      int    h_length;          /* length of address ,包含头长(8字节)*/
      char **h_addr_list;       /* list of addresses */
}

[gethostbyname()函数详解_带鱼兄的博客-CSDN博客_gethostbyname](https://blog.csdn.net/daiyudong2020/article/details/51946080)

代码：(注意windows上使用网络一定要开启WSA,不要在inet_ntoa内部进行强制类型转换，会报错)

```C++
#include<iostream>
#include<string>
#include<vector>
#include<WinSock2.h>
using namespace std;
int main()
{
	WSADATA ws;
	int ret = WSAStartup(MAKEWORD(2, 2), &ws);
	if (ret) return 0;
	string str = "www.baidu.com";
	hostent* host_entry = gethostbyname(str.c_str());
	cout << str.c_str() << endl;
	if (host_entry == NULL) {
		cout << "failed" << endl;
		exit(0);
	}
	char** fd = host_entry->h_addr_list;
	while (fd != NULL)
	{
		in_addr* ls = (in_addr*) *fd;
		cout << inet_ntoa(*ls)<<endl;
		fd++;
	}

	return 0;
}
```

注：UDP   socket创建是数据包发送格式采用数据报发送SOCK_DGRAM(数据报套接字/无连接的套接字)

​		TCP	socket创建采用SOCK_STREAM（流格式套接字/有连接的套接字）

### UDP和MTU的关系

sendto 最好发送1400字节

与TCP的不同：

报文不允许分段读，一次只能读一个报文

### UDP的缺点

udp在设计应用层协议时需要解决重传，丢包，乱序等问题，因此要求应用层协议需要有以下特点：

1.ACK机制

2.重传机制（重传时间RTO一般为往返时延RTT的1.5倍）

3.序号机制

4.重排机制

5.窗口机制

### 注意

udp具有小包优先传，即小于mtu的包先传的机制

