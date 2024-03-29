### 常见errorno值

```
connect:
EINPROGRESS(正在建立)
EISCONN(已经建立)
```

```
write：
EPIPE(消息未发送出去)
EWOULDBLOCK(写缓冲区为空)
EINTR(write操作被打断)
```

```
read：
EWOULDBLOCK(读缓冲区为空)
EINTR(read操作被打断)
```

```
EPOLLRDHUB(服务器读端关闭)
EPOLLHUB(服务器双端关闭)
```

