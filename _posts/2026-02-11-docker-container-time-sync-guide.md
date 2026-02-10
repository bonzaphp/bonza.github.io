---
layout: post
title:  "容器时间跟宿主机时间不一致解决方案"
date:   2026-02-11 02:11:00 +0800
categories: docker
tags: docker 容器 时间同步 时区 jenkins
author: 来财
---


* content
{:toc




本文详细介绍了 Docker 容器与宿主机时间不一致的问题及解决方案，包括共享主机 localtime、复制主机 localtime、自定义 Dockerfile 设置时区等多种方法，特别针对 Jenkins 构建时间错误提供了完整的解决方案。

## 容器时间跟宿主机时间不一致

***

### 在Docker容器创建好之后，可能会发现容器时间跟宿主机时间不一致，这就需要同步它们的时间，让容器时间跟宿主机时间保持一致。如下：

```bash
# 宿主机时间
[root@slave-1 ~]# date
Fri May 12 11:20:30 CST 2017
 
#容器时间
[root@slave-1 ~]# docker exec -ti 87986863838b /bin/bash
root@87986863838b:/# date                                                                                                                   
Fri May 12 03:20:33 UTC 2017
 
# 发现两者之间的时间相差了八个小时！
# 宿主机采用了CST时区，CST应该是指（China Shanghai Time，东八区时间）
# 容器采用了UTC时区，UTC应该是指（Coordinated Universal Time，标准时间）
 
# 统一两者的时区有下面几种方法
# 1）共享主机的localtime
# 创建容器的时候指定启动参数，挂载localtime文件到容器内，保证两者所采用的时区是#一致的。
# docker run -ti -d --name my-nginx -v /etc/localtime:/etc/localtime:ro  docker.io/nginx  /bin/bash
 
# 2)复制主机的localtime
[root@slave-1 ~]# docker cp /etc/localtime 87986863838b:/etc/
 
# 然后再登陆容器，查看时间，发现已经跟宿主机时间同步了
[root@slave-1 ~]# docker exec -ti 87986863838b /bin/bash
[root@87986863838b:/]# date  

Fri May 12 11:26:19 CST 2017
 
# 3）创建dockerfile文件的时候，自定义该镜像的时间格式及时区。在dockerfile文件里#添加下面内容：
......
FROM tomcat
ENV CATALINA_HOME /usr/local/tomcat
.......
# 设置时区

RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone
......
 
 

```

#### 保存后，利用docker build命令生成镜像使用即可,使用dockerfile创建的镜像的容器改变了容器的时区，这样不仅保证了容器时间与宿主机时间一致（假如宿主机也是CST）,并且像上面使用tomcat作为父镜像的话，JVM的时区也是CST,这样tomcat的日志信息的时间也是和宿主机一致的，像上面那两种方式只是保证了宿主机时间与容器时间一致，JVM的时区并没有改变，tomcat日志的打印时间依旧是UTC。

#### ==Jenkins构建时间错误 将时区设置为上海时间==

打开 【系统管理】->【脚本命令行】运行下面的命令

`System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'Asia/Shanghai')`

#### 我在k8s里起了一个jenkins项目，镜像使用的是官方的docker镜像`image: jenkins`,在使用过程中发现，jenkins的时区不对，使用的是0时区。然后我在网上找了很多方式，以为通过传递环境变量可以解决，但是都不可行。最终，我下载了官方镜像的`Dockerfile`来重新`build`，在build之前在Dockerfile里添加下列两行，解决

```bash
RUN rm -rf /etc/localtime && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

RUN echo 'Asia/Shanghai' > /etc/timezone
```

### 亲测有效

开启容器之前加上参数

    $ docker run ... -e JAVA_OPTS=-Duser.timezone=Asia/Shanghai

如果是docker-compose

可以加environment

```yml
...
 environment:
        JAVA_OPTS: "-Duser.timezone=Asia/Shanghai"
...
```