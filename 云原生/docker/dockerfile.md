[6.Dockerfile.pdf](file:///D:/零声Linux/docker/6.Dockerfile.pdf)

### dockerfile 关键字

```
FROM 设置镜像使用的基础镜像
MAINTAINER 设置镜像的作者
RUN 编译镜像时运行的脚本
CMD 设置容器的启动命令
LABEL 设置镜像标签
EXPOSE 设置镜像暴露的端口
ENV 设置容器的环境变量
ADD 编译镜像时复制上下文中文件到镜像中
COPY 编译镜像时复制上下文中文件到镜像中
ENTRYPOINT 设置容器的入口程序
VOLUME 设置容器的挂载卷
USER 设置运行 RUN CMD ENTRYPOINT的用户名
WORKDIR 设置 RUN CMD ENTRYPOINT COPY ADD 指令的工作目录
ARG 设置编译镜像时加入的参数
ONBUILD 设置镜像的ONBUILD 指令
STOPSIGNAL 设置容器的退出信号量
```

注意：

```
ADD ./helloworld.tar.gz ./  会自动解压，而COPY不会
ADD http://a.com//helloworld.tar.gz  ./ 只会下载，不会解压			
```

