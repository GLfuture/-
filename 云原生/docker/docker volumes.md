[7.Docker数据卷.pdf](file:///D:/零声Linux/docker/docker数据卷、网络、监控/7.Docker数据卷.pdf)

### 指定目录绑定数据卷

```
docker run -dit -v 宿主机目录(或已创建的数据卷):容器目录:ro(可读)
```

不指定宿主机目录会创建匿名数据卷