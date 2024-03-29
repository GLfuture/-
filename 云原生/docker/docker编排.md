[10. Docker Compose.pdf](file:///D:/零声Linux/docker/docker 容器编排与集群/10. Docker Compose.pdf)

### docker 运行mysql镜像

```
docker run -it -d --name mysql --net=host -v /root/mysql/data:/var/lib/mysql -v /root/mysql/config:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 -e TZ=Asia/Shanghai mysql
```

### docker swarm AND docker service

[11. Docker Swarm.pdf](file:///D:/零声Linux/docker/docker 容器编排与集群/11. Docker Swarm.pdf)

```
docker swarm join-token worker(manager)//获取token
```

```
docker swarm ca --rotate//更新证书
docker swarm update --autolock=true//会返回秘钥
docker swarm unlock-key//如果密匙忘了，可以通过这个命令返回密匙
```

leader node发生故障会重新选举一个manager node为leader，如果在docker swarm init时指定了--auto-lock，此时需要获取leader的key

锁住的manager node需要解锁

```
docker swarm unlock-key//返回密匙
docker swarm unlock//解锁
```

### 服务发布到私有注册中心

```
docker service create --name myhello1 -p 82:80 --replicas 5 --with-registry-auth 镜像
```

