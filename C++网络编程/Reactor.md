## Reactor(管理epoll)

就是epoll的封装

reactor的目的就是实现fd的分离，可以将io的属性封装在类中，每个io可以对应不同的回调函数\

reactor 的三种模型，见：redis(单线程reactor),memcached(多线程reactor)，nginx(多进 程reactor)

##### 下面的例子有不好的地方：

应该在调用send/write返回-1后再去注册写事件，而不是直接修改注册的事件，用写事件去判断是否send/write

下面通过reactor模型实现了网络io的处理

可以引入线程池，参照redis在业务处理时将解码，编码的任务抛给线程池。

#### sock.h

```c++
#pragma once
#include<iostream>
#include<sys/epoll.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<netinet/in.h>
#include<string>
#include<vector>
#include<memory>
#include<fcntl.h>
#include<string.h>
#define MAX_LISTEN_NUM 10
#define MAX_BLOCK_NUM 1024
#define MAX_EPOLL_RETURN_NUM 1024
#define MAX_BUFFER_SIZE 1024
using namespace std;
class item {
public:
	item(int m_client, uint32_t m_event, string m_wbuf, string m_rbuf)
	{
		this->clientfd = m_client;
		this->event = m_event;
		this->r_buffer = m_rbuf;
		this->w_buffer = m_wbuf;
	}
	int clientfd;
	string w_buffer;
	string r_buffer;
	uint32_t event;
    //下面会引起段错误，因为后面释放内存之前又调用了析构函数,同样使用智能指针时自动释放内存也会产生这个问题
	//~item()
	//{
	//	delete this;
	//}
};
class item_block {
public:
	item_block()
	{
		items.resize(MAX_BLOCK_NUM, NULL);
	}
	vector<item*> items;
};
class reactor {
public:
	reactor()
	{
		this->epollfd = epoll_create(1);
		this->fdcount = 0;
	}
	void Add_To_Reactor(int clientfd, uint32_t epoll_type);
	void DEL_FROM_Reactor(int clientfd, uint32_t epoll_type);
	void Deal_Events(int sock);
	void ADD_TO_EPOLL(int clientfd, uint32_t epoll_type);
	void DELETE_FROM_EPOLL(int clientfd, uint32_t epoll_type);
	int epollfd;
	int fdcount;//fd数量
	vector<item_block> blocks;

};
```

#### scok.cpp:

```c++
#include"sock.h"
int Init_Sock(int port)
{
	int sockfd = socket(AF_INET, SOCK_STREAM, 0);
	if (sockfd <= 0) {
		exit(0);
	}
	cout << sockfd << endl;
	sockaddr_in sin;
	sin.sin_addr.s_addr = htonl(INADDR_ANY);
	sin.sin_family = AF_INET;
	sin.sin_port = htons(port);
	if (bind(sockfd, (sockaddr*)&sin, sizeof(sin)) == -1)
	{
		cout << "bind faild" << endl;
		exit(0);
	}
	int flag = fcntl(sockfd, F_GETFL, 0);
	flag |= O_NONBLOCK;
	fcntl(sockfd, F_SETFL, flag);
	listen(sockfd, MAX_LISTEN_NUM);
	return sockfd;
}

void reactor::Add_To_Reactor(int clientfd, uint32_t epoll_type)
{
	long blcs_ind = clientfd / MAX_BLOCK_NUM;
	long blc_ind = clientfd % MAX_BLOCK_NUM;
	item* blc = new item(clientfd, epoll_type, "", "");
	if ((this->blocks.size() == 0) || blcs_ind > (this->blocks.size() - 1)) {
		item_block newblock;
		this->blocks.push_back(newblock);
	}
	this->blocks[blcs_ind].items[blc_ind] = blc;
	this->fdcount++;
	ADD_TO_EPOLL(clientfd, epoll_type);
}

void reactor::DEL_FROM_Reactor(int clientfd, uint32_t epoll_type)
{
	int blcs_ind = clientfd / MAX_BLOCK_NUM;
	int blc_ind = clientfd % MAX_BLOCK_NUM;
	DELETE_FROM_EPOLL(clientfd, epoll_type);
	close(clientfd);
	delete this->blocks[blcs_ind].items[blc_ind];
	this->blocks[blcs_ind].items[blc_ind]=NULL;
}

void reactor::Deal_Events(int sock)
{
	epoll_event events[MAX_EPOLL_RETURN_NUM];
	char* buffer = new char[MAX_BUFFER_SIZE];
	while (1)
	{
		int nready = epoll_wait(this->epollfd, events, MAX_EPOLL_RETURN_NUM, -1);
		if (nready == -1) continue;
		for (int i = 0; i < nready; i++)
		{
			if (events[i].data.fd == sock)
			{
				sockaddr_in sin;
				socklen_t len = sizeof(sin);
				int clientfd = accept(sock, (sockaddr*)&sin, &len);
				this->Add_To_Reactor(clientfd, EPOLLIN);
				int flag = fcntl(sock, F_GETFL, 0);
				flag |= O_NONBLOCK;
				fcntl(sock, F_SETFL, flag);
			}
			else {
				int clientfd = events[i].data.fd;
				int blcs_ind = events[i].data.fd / MAX_BLOCK_NUM;
				int blc_ind = events[i].data.fd % MAX_BLOCK_NUM;
				epoll_event ev;
				ev.data.fd = events[i].data.fd;
				item* it = this->blocks[blcs_ind].items[blc_ind];
				//it = move(this->blocks[blcs_ind].items[blc_ind]);
				if (events[i].events & EPOLLIN)
				{
					int rlen = recv(clientfd, buffer, MAX_BUFFER_SIZE, 0);
					if (rlen <= 0) {
						DEL_FROM_Reactor(clientfd, EPOLLIN);
						cout << "connect close" << endl;
					}
					else {
						it->r_buffer = buffer;
						it->w_buffer = buffer;
						cout << buffer << endl;
						ev.events = EPOLLOUT;
						epoll_ctl(this->epollfd, EPOLL_CTL_MOD, clientfd, &ev);
						memset(buffer, 0, MAX_BUFFER_SIZE);
					}
				}
				else if (events[i].events & EPOLLOUT)
				{
					int wlen = send(clientfd, it->w_buffer.c_str(), it->w_buffer.length(), 0);
					it->w_buffer = "";
					it->r_buffer = "";
					ev.events = EPOLLIN;
					epoll_ctl(this->epollfd, EPOLL_CTL_MOD, clientfd, &ev);
					if (wlen <= 0) {
						cout << "send failed" << endl;
					}
				}
			}
		}
	}
}

void reactor::ADD_TO_EPOLL(int clientfd, uint32_t epoll_type)
{
	epoll_event ev;
	ev.data.fd = clientfd;
	ev.events = epoll_type;
	epoll_ctl(this->epollfd, EPOLL_CTL_ADD, clientfd, &ev);
}

void reactor::DELETE_FROM_EPOLL(int clientfd, uint32_t epoll_type)
{
	epoll_event ev;
	ev.data.fd = clientfd;
	ev.events = epoll_type;
	epoll_ctl(this->epollfd, EPOLL_CTL_DEL, clientfd, &ev);
}

```

#### main.cpp:

```c++
#include"sock.h"

int main(int argc, char* argv[])
{
	int sockfd = Init_Sock(atoi(argv[1]));//创建sock
	reactor* R = new reactor();
	R->Add_To_Reactor(sockfd, EPOLLIN);
	R->Deal_Events(sockfd);
	return 0;
}

```

