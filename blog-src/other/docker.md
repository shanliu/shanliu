# docker

> 镜像操作

```
#编译 -t 名:tag -f 指定Dockerfile文件
#docker build -t php:1 -f ./docker/yp.Dockerfile ./docker/
#查看镜像
#docker images
#删除镜像
#docker rmi 772c531cd8ea[id]
#从容器中创建镜像
#docker commit -a "test" -m "test" a404c6c174a2 php:1
```

> 容器操作

```
#创建 -v 本地:容器 -d 后台执行 -P 当存在EXPOSE时 -p 端口映射
#docker create -p 80:80 -v /data:/data --name mp php
#运行 停止重启 stop restart
#docker start mp
#查看容器列表 -n 5 限制数量
#docker ps -a
#查看指定容器信息 (镜像可用)
#docker inspect mp
#删除 -v 连带删除数据 -f 强制停止 -l 移除数据连接
#docker rm mp
#从新进入容器/bin/sh
#docker exec -it mp /bin/sh
#创建并运行容器 create && start
#docker run -d nginx:latest
#以交互方式启动容器 --rm 退出后删除
#docker run --rm -it ubuntu bash
#kubectl
#kubectl -- run --rm -i --tty my-image-pod --image=debian:buster-slim --restart=Never -- /bin/bash
#KILL容器
#docker kill mp
#容器关联 容器内网络相互连通 --link
docker run --rm -it --link mongo -p 9001:9000 php
#环境变量 --env 
docker create -p 8080:8080 --env URLS="[ { url: 'http://petstore.swagger.io/v2/swagger.json', name: 'TEST1'} ]" --name swagger_dome swaggerapi/swagger-ui
```


> 本地镜像服务

```
#docker run -d -v /registry:/var/lib/registry -p 5000:5000 --restart=always --privileged=true --name registry registry:latest
#客服端修改 /etc/default/docker
#DOCKER_OPTS="$DOCKER_OPTS --registry-mirror=https://registry.docker-cn.com --insecure-registry=192.168.1.250:5000"
#打指定服务器的TAG后上传
#docker tag ubuntu 192.168.1.250:5000/ub1
#docker push 192.168.1.250:5000/ub1
#其他服务器拉取
#docker pull 192.168.1.250:5000/ub1
```

>PHP镜像示例

```
FROM alpine:3.8
#指定作者
#MAINTAINER aaa
​
#指定标签
LABEL maintainer="lonely" version="1.0"
​
#设置 RUN, CMD 和 ENTRYPOINT 的运行用户
#USER daemo
​
​
#暴露端口
#EXPOSE 80
​
#设置环境变量
#ENV <key>=<value>
​
#拷贝本地文件到指定路径
#ADD <src>... <dest> 支持网络
#COPY <src>... <dest> 只支持本地
​
​
#构件容器时运行
#RUN ls
​
#启动时执行 同时存在cmd 和 ENTRYPOINT 将忽略 ENTRYPOINT 或 cmd 为参数
#ENTRYPOINT ["executable", "param1", "param2"]
#CMD ls -al
​
#挂载目录 目录自身挂载 无法指定
#VOLUME ["/data1","/data2"]
​
#设置工作目录 类似cd
#WORKDIR /path/to/workdir
​
#定义变量
#ARG <name>[=<default value>]
ARG user1=someuser
#功能为容器启动时要运行的命令
​
#容器退出系统发送信号
#STOPSIGNAL INT
​
#子镜像构建时执行
#ONBUILD [INSTRUCTION]
​
#容器健康状况检查命令
#0: success - 表示容器是健康的
#1: unhealthy - 表示容器已经不能工作了
#2: reserved - 保留值
#HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1
​
​
ARG timezone
# prod pre test dev
ARG app_env=prod
# default use www-data user
ARG add_user=www-data
​
ENV APP_ENV=${app_env:-"prod"} \
TIMEZONE=${timezone:-"Asia/Shanghai"}
​
### ---------- building ----------
​
RUN set -ex \
&& sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/' /etc/apk/repositories \
&& apk update \
&& apk add --no-cache \
nginx \
ca-certificates \
curl \
tar \
xz \
libressl \
tzdata \
pcre \
php7 \
php7-bcmath \
php7-curl \
php7-ctype \
php7-dom \
php7-fileinfo \
php7-gettext \
php7-gd \
php7-iconv \
php7-json \
php7-mbstring \
php7-mongodb \
php7-mysqlnd \
php7-openssl \
php7-pdo \
php7-pdo_mysql \
php7-pdo_sqlite \
php7-phar \
php7-posix \
php7-redis \
php7-simplexml \
php7-sockets \
php7-sodium \
php7-session \
php7-sysvshm \
php7-sysvmsg \
php7-sysvsem \
php7-tokenizer \
php7-zip \
php7-zlib \
php7-fpm \
&& apk del --purge *-dev \
&& rm -rf /var/cache/apk/* /tmp/* /usr/share/man /usr/share/php7 \
&& cd /etc/php7 \
&& { \
echo "upload_max_filesize=100M"; \
echo "post_max_size=108M"; \
echo "memory_limit=1024M"; \
echo "date.timezone=${TIMEZONE}"; \
} | tee conf.d/99-overrides.ini \
&& { \
echo "[global]"; \
echo "pid = /var/run/php-fpm.pid"; \
echo "[www]"; \
echo "user = www-data"; \
echo "group = www-data"; \
} | tee php-fpm.d/custom.conf \
&& chown -R www-data:www-data /var/www \
&& { \
echo "#!/bin/sh"; \
echo "nginx -g 'daemon on;'"; \
echo "php-fpm7 -F"; \
} | tee /run.sh \
&& chmod 755 /run.sh \
&& ln -sf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
&& echo "${TIMEZONE}" > /etc/timezone \
&& addgroup -g 82 -S ${add_user} \
&& adduser -u 82 -D -S -G ${add_user} ${add_user} \
&& mkdir -p /data \
&& chown -R ${add_user}:${add_user} /data
​
EXPOSE 80
VOLUME ["/var/www", "/data"]
WORKDIR /var/www
​
CMD /run.sh
```
