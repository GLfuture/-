### 配置

[Kong 动态负载均衡-2207-VIP.pdf](file:///D:/零声Linux/openresty/Kong 动态负载均衡-2207-VIP.pdf)

# Kong 简介 

> Kong 是一款基于 openresty 编写的高可用、易扩展的开源 API Gateway 项目。
>
> Kong 支持两种工作模式：一种是不使用数据库；另一种是使用
>
> 数据库；可用的数据库为 PostgreSQL、Cassandra（分布式 NoSQL 数据库）

# Kong 参考文档 

> 官方网站：[[https://kon]{.ul}g[hq.com]{.ul}](https://konghq.com/) 
>
> 官方文档：[[https://docs.kon]{.ul}g[hq.com]{.ul}](https://docs.konghq.com/) 
>
> 项目地址：[[https://]{.ul}g[ithub.com/Kon]{.ul}g[/kon]{.ul}g](https://github.com/Kong/kong)
>
> 中文文档：[[https://]{.ul}g[ithub.com/wan]{.ul}g[lon]{.ul}g[sxr/kon]{.ul}g[-docs-cn]{.ul}](https://github.com/wanglongsxr/kong-docs-cn) 基于1.1.x版本

# docker 安装（推荐） 

[(106条消息) 以Docker方式安装和配置Kong网关和Konga控制台_nklinsirui的博客-CSDN博客](https://blog.csdn.net/nklinsirui/article/details/118892872)

## 创建 Kong 网络 

创建自定义 Docker 网络以允许容器相互发现和通信

```
sudo docker network create kong-net
```

## 安装 PostgreSQL 

使用postgresql

[PostgreSQL教程（一）：从头开始-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/173470#:~:text=PostgreSQL教程（一）：从头开始 1 1.1. 安装 自然，在你能开始使用PostgreSQL之前， 你必须安装它。 PostgreSQL很有可能已经安装到你的节点上了， 因为它可能包含在你的操作系统的发布里，,... 4 1.4. 访问数据库 一旦你创建了数据库，你就可以通过以下方式访问它： 运行PostgreSQL的交互式终端程序，它被称为psql， 它允许你交互地输入、编辑和执行SQL命令。 )

 创建 PostgreSQL 容器

```
docker run -d --name kong-database \
               --network=kong-net \
               -p 5432:5432 \
               -v $HOME/kong/postgres-data:/var/lib/postgresql/data \
               -e "POSTGRES_USER=kong" \
               -e "POSTGRES_DB=kong" \
               -e "POSTGRES_PASSWORD=kong" \
               postgres:9.6
```

###  使用 Kong 容器运行进行数据库初始化

``` 
docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_USER=kong" \
     -e "KONG_PG_PASSWORD=kong" \
     kong:2.5.0 kong migrations bootstrap
```

#### 1.对kong admin api作安全防护

在$HOME/kong/config目录下编写kong.yml

```
_format_version: "1.1"

services:
- name: admin-api
  url: http://127.0.0.1:8001
  routes:
    - paths:
      - /admin-api
  plugins:
  - name: key-auth

consumers:
- username: admin
  keyauth_credentials:
  - key: secret
```

导入kong配置：

```
docker run --rm     --network=kong-net     -e "KONG_DATABASE=postgres"     -e "KONG_PG_HOST=kong-database"     -e "KONG_PG_USER=kong"     -e "KONG_PG_PASSWORD=kong"     -v $HOME/kong/config:/home/kong     kong:2.5.0 kong config db_import /home/kong/kong.yml
```

#### 2.不对kong做api防护(直接创建kong容器)

```
sudo docker run -d --name kong --network=kong-net -u root -e "KONG_DATABASE=postgres" -e  "KONG_PG_HOST=kong-database" -e "KONG_PG_USER=kong" -e "KONG_PG_PASSWORD=kong" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" -e "KONG_PROXY_ERROR_LOG=/dev/stderr" -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" -e "KONG_ADMIN_LISTEN=0.0.0.0:8001,0.0.0.0:8444 ssl" -p 8000:8000 -p 8443:8443 -p 8001:8001 -p 8444:8444 --restart always kong:2.5.0
```

## 创建 Kong 容器 (client端口8000,server端口8001)

### -u -root: Kong 启动过程中，需要权限创建一些文件和文件夹；

```
docker run -d --name kong \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_USER=kong" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
     -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
     -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
     -p 8000:8000 \
     -p 8443:8443 \
     -p 127.0.0.1:8001:8001 \
     -p 127.0.0.1:8444:8444 \
     kong:2.5.0
```

[kong启动后外部主机无法访问8001管理端口_kong 8001_Min_Monk的博客-CSDN博客](https://blog.csdn.net/Min_Monk/article/details/104083839/)

## 搭建 Konga (端口;1337)

> Konga 是 Kong 的可视化 API 操作工具；
>
> [[https://hub.docker.com/r/pantsel/kon]{.ul}g[a/]{.ul}](https://hub.docker.com/r/pantsel/konga/)

###  复用上面的数据库容器

```
docker run -d --name konga-database \
               --network=kong-net \
               -p 5433:5432 \
               -v $HOME/kong/konga/postgres-data:/var/lib/postgresql/data \
               -e "POSTGRES_USER=konga" \
               -e "POSTGRES_DB=konga" \
               -e "POSTGRES_PASSWORD=konga" \
               postgres:9.6
```

运行 Konga 容器

#### 初始化数据库

```
docker run --rm \
             --network=kong-net \
             pantsel/konga:latest \
             -c prepare \
             -a "postgres" \
             -u "postgres://konga:konga@konga-database:5432/konga"

```

#### 启动konga

```
docker run -d --name konga \
             --network kong-net \
             -e "TOKEN_SECRET=secret123" \
             -e "DB_ADAPTER=postgres" \
             -e "DB_URI=postgres://konga:konga@konga-database:5432/konga" \
             -e "NODE_ENV=development" \
             -p 1337:1337 \
             pantsel/konga
```

### 插件

kong的插件只能用在三个对象(service、route、consumer)



