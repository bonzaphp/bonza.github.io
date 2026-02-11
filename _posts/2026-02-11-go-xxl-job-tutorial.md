---
layout: post
title:  "Go 语言使用定时调度任务大杀器 — XXL-JOB"
date:   2026-02-11 22:04:55 +0800
categories: golang
tags: golang xxl-job 定时任务 分布式 调度
author: 来财
---

XXL-JOB 是一个分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。本文将以一个 gopher 的视角教你跳过 java 环境，做到仅仅使用 golang 也可以开箱即用，玩转 XXL-JOB。

* content
{:toc}





# Go 语言使用定时调度任务大杀器 — XXL-JOB

> [XXL-JOB](https://github.com/xuxueli/xxl-job) 是一个分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。
>
> 本文将以一个 gopher 的视角教你跳过 java 环境，做到仅仅使用 golang 也可以开箱即用，玩转 XXL-JOB。
>
> XXL-JOB [使用文档](https://www.xuxueli.com/xxl-job/)

## 为什么要用

XXL- JOB 在我看来可以替代我们系统中所有定时任务调度，因为定时任务的不可掌控，尤其是在分布式集群中。如果一个任务失败了那么我们首先无法即时发现，其次我们就算发现了也没法去重新触发他来解决问题，解决问题只能靠重启服务，但有的服务又不能随便重启，打个比方：行情服务，他一直在不断的接收数据流，重启势必造成数据丢失，丢失都还是好的重启服务还会造成数据堆积，消息队列阻塞，数据更新延迟，所以一旦这种服务的定时任务出现问题你只有任他发展。这个时候 XXL- JOB 这个应用就成了解决问题的大杀器。

*   定时及手动调度支持

它帮你承担了调度的任务，支持 cron 方式定义任务的定时调度，同时支持手动出发任务。

*   多种任务注册模式

主要有 Bean 和 GLUE 两种模式其中GLUE 模式支持java ,shell, pyhthon, PHP, PowerShell, NodeJS 等几种语言，任务代码在控制台定义由控制台存储管理，执行代码在执行器中，实现方法就是先在执行器中编译并形成脚本然后按照调度定义来触发。这个模式我们不做介绍感觉用处不大。Bean模式支持HTTP 和 命令 调用。本文只介绍 HTTP 调用。

*   界面化管理

同时支持可视化，调度信息界面化编辑管理，包括任务新建，更新，删除，GLUE开发和任务报警等，所有上述操作都会实时生效，同时支持监控调度结果以及执行日志，支持执行器Failover。

*   分布式系统支持

XXL- JOB 支持分布式系统，你可以在你的分布式服务上和单体服务一样的注册执行器，只要你保证集群内每个服务注册的执行器名称一致即可， XXL- JOB的调度中心会将同名执行器地址都记录下来，在选择执行器时只选择一个执行。

这样一个可以帮你管理监控定时任务，并在失败或是需要时手动触发是不是解决你的很多痛点，叫他大杀器一点咩有错吧。还不跟我一起用起来！！！

## 搭建 XXL- JOB

本文的目标就是让 gopher 们跳过 JAVA 环境来从0到1的学习并使用 XXL - JOB 。要学习使用 XXL - JOB 首先要本地搭建一个平台，那么如何跳过 JAVA 来搭建呢，你肯定想到了用 docker ，对的肯定是要用容器，而且作者本身也在开源镜像库中给大家准备了这个镜像，但你要是想要使用最新版本的代码呢？或者你就想用更老版本的代码，这个时候你必须自己编译代码生成 jar 包，环境这块是我最烦的所以我经过不断试错给大家准备了一个可以直接编译源码并形成即刻使用的镜像的 Dockerfile ，请叫我雷锋。

你只要跟我下面说的一步步操作，就可以跳过JAVA 环境直接搭建起来一个 XXL- JOB 平台：

### 1. 克隆我的可直接构建镜像代码

我的代码在[这里](https://github.com/LittleBeeMark/xxl-job)

这个代码是可以直接构建镜像并直接跑起来的，我直接fork了作者的代码并修改了配置文件，添加了 Dockerfile 和 Makefile 。

下面是我的 Dokerfile 和 Makefile 我注释都写的很清楚就不去解释了，其实就是使用了docker的多阶段镜像构建的功能，实现了先编译再运行的目标。

```


### 第一阶段，用maven镜像进行编译
FROM maven:3.6.3 AS compile_stage

####################定义环境变量 start####################
#定义工程名称，也是源文件的文件夹名称
ENV PROJECT_NAME xxl-job
#定义工作目录
ENV WORK_PATH /usr/src/$PROJECT_NAME

####################定义环境变量 start####################
#将源码复制到当前目录
COPY . $WORK_PATH

#编译构建
RUN cd $WORK_PATH && mvn clean package -Dmaven.test.skip

### 第二阶段，用第一阶段的jar和jre镜像合成一个小体积的镜像
FROM openjdk:8-jre-slim

####################定义环境变量 start####################
#定义工程名称，也是源文件的文件夹名称
ENV PROJECT_NAME xxl-job/xxl-job-admin
#定义工作目录
ENV WORK_PATH /usr/src/$PROJECT_NAME

####################定义环境变量 start####################
MAINTAINER xuxueli
#工作目录是/opt
WORKDIR /opt

#从名为compile_stage的stage复制构建结果到工作目录
COPY --from=compile_stage $WORK_PATH/target/xxl-job-admin-*.jar  /opt/app.jar

ENV PARAMS=""

ENV TZ=PRC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

#启动应用
ENTRYPOINT ["/usr/local/openjdk-8/bin/java","-jar","/opt/app.jar"]
SERVICE_NAME=ddcxxljobadmin
SERVICE_VERSION=v1.0.0

docker_build:
 docker build -t $(SERVICE_NAME):$(SERVICE_VERSION) ./
```

Makefile 就是一个 docker\_build 命令来构建镜像。

### 2. 在本地数据库执行脚本建表

代码中[这里 ](https://github.com/LittleBeeMark/xxl-job/tree/master/doc/db)tree/master/doc/db 目录下有一个 [tables\_xxl\_job.sql](https://github.com/LittleBeeMark/xxl-job/blob/master/doc/db/tables_xxl_job.sql) 文件，你需要现在你本地数据库（注意，这里需要的是Mysql数据库）中执行这个文件，当然你可以在任何数据中执行，但你本地要可以连得上，因为这是我们一会要搭建的 XXL- JOB 平台需要依赖的数据库。

执行好后你会发现你的数据库中多了 xxl-job 这个库，结构如下图：

[![image-20221014151029839](https://marksuper.xyz/picture/file/xxl-job/xxl-job-1.png)](https://marksuper.xyz/picture/file/xxl-job/xxl-job-1.png)

### 3. 修改配置

我的代码中已经将数据库地址改为了本地你可以改为的数据部署IP，配置文件地方在[这里](https://github.com/LittleBeeMark/xxl-job/blob/master/xxl-job-admin/src/main/resources/application.properties) [xxl-job](https://github.com/LittleBeeMark/xxl-job)/[xxl-job-admin](https://github.com/LittleBeeMark/xxl-job/tree/master/xxl-job-admin)/[src](https://github.com/LittleBeeMark/xxl-job/tree/master/xxl-job-admin/src)/[main](https://github.com/LittleBeeMark/xxl-job/tree/master/xxl-job-admin/src/main)/[resources](https://github.com/LittleBeeMark/xxl-job/tree/master/xxl-job-admin/src/main/resources)/ 目录下的 **application.properties** 文件你需要修改：

```MAKEFILE

### xxl-job, datasource
spring.datasource.url=jdbc:mysql://host.docker.internal:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=12345678
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

这里的 host.docker.internal 其实就是容器去访问的宿主机的 127.0.0.1 你如果也是本地启动的mysql你就不要改这个，你只要改下用户名和密码就好。

还有一点要说明的是：

```makefile

### web
server.port=8080
server.servlet.context-path=/xxl-job-admin
```

这个配置是启动 xxl-job-admin 管理平台的端口 8080 和路由 /xxl-job-admin 不需要改他，但要知道因为一会儿启动镜像时要把这个8080 端口映射出去。

### 4. 构建镜像

刚刚克隆好的代码 cd 进入到你刚刚克隆代码的 xxl-job 项目下，执行以下命令：

```

make docker_build
```

刚刚也看了我的 makefile 文件这个命令就是去构建镜像的。

执行成功后 使用docker images 就可以看到这个镜像：

    ddcxxljobadmin                v1.0.0                    7e7eb87cccd9   24 hours ago   237MB

这里的构建时间非常久。。。不要放弃，成功就在20分钟后，哈哈哈。

### 5. 跑起镜像

构建好镜像后使用：

    docker run -p 8090:8088 ddcxxljobadmin:v1.0.0

来把镜像跑起来。我这里把端口映射到了 8090 你可以映射到你需要的端口。

最后访问 <http://127.0.0.1:8090/xxl-job-admin/> 你会看到这个界面：

[![image-20221014152834734](https://marksuper.xyz/picture/file/xxl-job/xxl-job-2.png)](https://marksuper.xyz/picture/file/xxl-job/xxl-job-2.png)

恭喜你成功了🎉🎉

## 用 go 语言使用 XXL-JOB

我们接下来要使用 go 语言来使用这个平台，就是注册执行器，并构建执行任务，最后使用这个平台来调控执行这个任务。

### 需要使用的包：[xxl-job-executor-go](https://github.com/xxl-job/xxl-job-executor-go)

这个 xxl-job-executor-go 你可以理解为一个 SDK ，我们通过这个包可以很方便的注册执行器和任务。

### 交互原理

因为我们主要介绍如何使用所以我们不去过多的看源码，但我觉得必要的交互原理还是要知道的，不知道 执行器和任务是如何注册进入 xxl-job ，xxl-job 又是如何去触发注册任务的感觉就像是盲人摸象，我们只了解下我们的服务和 xxl-job 是如何交互的。

刚刚我说了本文只做 http 模式的介绍，这个模式忽略语言，且也是 xxl-job-executor-go 这个包使用的注册执行器模式，我们先看下它的注册执行器的源码：

```go
func (e *executor) Run() (err error) {
 // 创建路由器
 mux := http.NewServeMux()
 // 设置路由规则
 mux.HandleFunc("/run", e.runTask)
 mux.HandleFunc("/kill", e.killTask)
 mux.HandleFunc("/log", e.taskLog)
 mux.HandleFunc("/beat", e.beat)
 mux.HandleFunc("/idleBeat", e.idleBeat)
 // 创建服务器
 server := &http.Server{
 Addr:         e.address,
 WriteTimeout: time.Second * 3,
 Handler:      mux,
 }
 // 监听端口并提供服务
 e.log.Info("Starting server at " + e.address)
 go server.ListenAndServe()
 quit := make(chan os.Signal)
 signal.Notify(quit, syscall.SIGKILL, syscall.SIGQUIT, syscall.SIGINT, syscall.SIGTERM)
 <-quit
 e.registryRemove()
 return nil
}
```

源码中这个执行器Run方法是注册执行器的方法，我们可以看到它去进行了http路由注册和监听，好了结合我刚刚说的http 注册模式，是不是你已经理解的差不多了，其实就是按照 xxl-job 规定的规则去实现了 http 的接口。在xxl-job调度器那边就是根据不同的指令去调用这个执行器的 http 接口比如我定义一个任务，要求他没秒钟调用一次，那么xxl-job 不会管你到底是什么执行逻辑他只在每秒钟去触发一下<http://address:port/run> + 任务名参数。只要两边定义的任务名一致即可成功，整个过程中肯定包含很多状态回掉之类的这个我们不需要了解，我们只要大致了解交互机制即可，最后为了验证我们的猜想我们再看下这个 /run 路由对应的 e.runTask 方法：

    //运行一个任务
    func (e *executor) runTask(writer http.ResponseWriter, request *http.Request) {
     e.mu.Lock()
     defer e.mu.Unlock()
     req, _ := ioutil.ReadAll(request.Body)
     param := &RunReq{}
     err := json.Unmarshal(req, &param)
     if err != nil {
     _, _ = writer.Write(returnCall(param, 500, "params err"))
     e.log.Error("参数解析错误:" + string(req))
     return
     }
     e.log.Info("任务参数:%v", param)
     if !e.regList.Exists(param.ExecutorHandler) {
     _, _ = writer.Write(returnCall(param, 500, "Task not registered"))
     e.log.Error("任务[" + Int64ToStr(param.JobID) + "]没有注册:" + param.ExecutorHandler)
     return
     }

     //阻塞策略处理
     if e.runList.Exists(Int64ToStr(param.JobID)) {
     if param.ExecutorBlockStrategy == coverEarly { //覆盖之前调度
     oldTask := e.runList.Get(Int64ToStr(param.JobID))
     if oldTask != nil {
     oldTask.Cancel()
     e.runList.Del(Int64ToStr(oldTask.Id))
     }
     } else { //单机串行,丢弃后续调度 都进行阻塞
     _, _ = writer.Write(returnCall(param, 500, "There are tasks running"))
     e.log.Error("任务[" + Int64ToStr(param.JobID) + "]已经在运行了:" + param.ExecutorHandler)
     return
     }
     }

     cxt := context.Background()
     task := e.regList.Get(param.ExecutorHandler)
     if param.ExecutorTimeout > 0 {
     task.Ext, task.Cancel = context.WithTimeout(cxt, time.Duration(param.ExecutorTimeout)*time.Second)
     } else {
     task.Ext, task.Cancel = context.WithCancel(cxt)
     }
     task.Id = param.JobID
     task.Name = param.ExecutorHandler
     task.Param = param
     task.log = e.log

     e.runList.Set(Int64ToStr(task.Id), task)
     go task.Run(func(code int64, msg string) {
     e.callback(task, code, msg)
     })
     e.log.Info("任务[" + Int64ToStr(param.JobID) + "]开始执行:" + param.ExecutorHandler)
     _, _ = writer.Write(returnGeneral())
    }

其实就是通过 xxl-job 请求的任务名称获取并调用了之前注册好的函数方法，执行完成后再去回掉xxl-job告诉他执行结果。

是不是和我们的猜想一样，再看一下任务注册方法就可以更加确定这个交互过程：

```go
// RegTask 注册任务
func (e *executor) RegTask(pattern string, task TaskFunc) {
 var t = &Task{}
 t.fn = task
 e.regList.Set(pattern, t)
 return
}
```

没有什么问题，就是将任务名称和函数签名相互呼应然后存储。

### 注册第一个任务

接下来我们注册我们的第一执行器和第一个任务，我的源代码在[这里](https://github.com/CoolGoPkg/apply_xxl_job)。

#### 跑起来我们的执行器服务

我们直接看代码，使用该包注册执行器和任务的代码：

```go

func StartXXLJobClient(conf *conf.DemoConfig) {

 exec := xxl.NewExecutor(
 xxl.RegistryKey(conf.XXLJobConf.AppName), //执行器名称
 xxl.ServerAddr(conf.XXLJobConf.AdminAddress),
 xxl.AccessToken(conf.XXLJobConf.Token),                     //请求令牌(默认为空)
 xxl.ExecutorPort(strconv.Itoa(conf.XXLJobConf.ClientPort)), //默认9999（非必填）
 xxl.SetLogger(&logger{}),                                   //自定义日志
 //xxl.ExecutorIp("127.0.0.1"),      //可自动获取
 )
 exec.Init()
 //设置日志查看handler
 exec.LogHandler(func(req *xxl.LogReq) *xxl.LogRes {
 return &xxl.LogRes{Code: 200, Msg: "", Content: xxl.LogResContent{
 FromLineNum: req.FromLineNum,
 ToLineNum:   2,
 LogContent:  "这个是自定义日志handler",
 IsEnd:       true,
 }}
 })
 //注册任务handler
 exec.RegTask("TestJob1", job.TestJob1)

 log.Fatal(exec.Run())
}
```

这个备注作者已经写的很清楚了，我是把这个配置写到了配置文件里，这样更加贴近大家的实战项目一些。

RegistryKey 方法就是注册了一个执行器的名称，这个名称之后要再 xxl-job-admin 中进行新建执行器后使用，你在注册执行器后再xxl-job中并不会显示需要你去新建一个执行器。

ServerAddr 写入的是执行状态回掉地址，其实就是我们的 xxl-job 管理平台的访问地址 <http://127.0.0.1:8090/xxl-job-admin> 作者贴心的把回掉地址和这个访问地址统一了。

AccessToken 这个 token 是可以定义的，如果你没有改配置文件那么就写default\_token ,这个必须写人。

ExecutorPort 执行器的监听端口。

ExecutorIp 执行器所在服务的 IP 这个可以不写会自动获取。

接下来就是注册任务 RegTask ，然后Run 起来

我们看下我注册的任务TestJob1

```go

func TestJob1(ctx context.Context, param *xxl.RunReq) (msg string) {
 fmt.Println("test one task" + param.ExecutorHandler + " param：" + param.ExecutorParams + " log_id:" + xxl.Int64ToStr(param.LogID))
 return "test done"
}
```

这个其实就是作者的 test 很简单就是打印一些东西。

我们看下main函数：

```go
func main() {
 conf.InitConf("conf/config.yaml")
 xxl_job.StartXXLJobClient(conf.Config)
}
```

好我们跑起来：

[![image-20221014172710020](https://marksuper.xyz/picture/file/xxl-job/xxl-job-3.png)](https://marksuper.xyz/picture/file/xxl-job/xxl-job-3.png)

这样说明我们已经注册执行器成功了。

#### xxl-job 新建执行器和任务

我们只是跑起来了任务执行器，并向 xxl-job 发起了执行器注册。那么要想xxl-job界面化控制该执行器来执行其注册的不同任务还需要在xxl-job-admin 上进行执行器和任务规则的新增。

新增执行器：

[![image-20221014173226569](https://marksuper.xyz/picture/file/xxl-job/xxl-job-4.png)](https://marksuper.xyz/picture/file/xxl-job/xxl-job-4.png)

在 xxl-job-admin 中的 ->执行器管理界面 -> 新增

在如上图的对话框中填入执行器的名称，注意这里的 AppName 而这里的名称无所谓你填什么他只起到展示作用。一定要填写和我们之前注册执行器服务的名称一致 ，注册方式自动获取，xxl-job就会 去自动获取我们注册的这个对应名称执行器的 IP 和 端口。

新增任务:

[![image-20221014175217984](https://marksuper.xyz/picture/file/xxl-job/xxl-job-5.png)](https://marksuper.xyz/picture/file/xxl-job/xxl-job-5.png)

在 xxl-job-admin 中的 -> 任务管理界面 -> 新增

这个任务的执行器选择刚刚我们新建的执行器，任务描述和负责人随意，运行模式使用BEAN模式，这个我们上面讲过了。JobHandler 这个参数很重要，这个需要填写和我们执行器中注册的任务一致，刚刚我们注册的第一个任务就是 TestJob1 所以这里我们也写这个最后点击保存。

恭喜你第一个任务就注册完成了！！

#### 执行，启动我们的任务

点击我们刚刚注册的任务的操作按钮，在下拉列表中点击执行一次。任务参数填写abc。看一下我们的执行器日志打印了什么：

    自定义日志 - 任务参数:&{2 TestJob1 abc SERIAL_EXECUTION 0 7 1665741822402 BEAN  1665718703000 0 1}
    自定义日志 - 任务[2]开始执行:TestJob1
    test one taskTestJob1 param：abc log_id:7

我们看到我们的ExecutorHandler 就是任务注册名称TestJob1， ExecutorParams 字段就是我们传入的参数 abc。恭喜你成功执行了该任务。

之后点击操作中的启动按钮，看看是否是一秒钟执行一次。

```shell
自定义日志 - 任务回调成功:{"code":200,"msg":null,"content":null}
自定义日志 - 任务参数:&{2 TestJob1  SERIAL_EXECUTION 0 19 1665742167004 BEAN  1665718703000 0 1}
自定义日志 - 任务[2]开始执行:TestJob1
test one taskTestJob1 param： log_id:19
自定义日志 - 任务回调成功:{"code":200,"msg":null,"content":null}
自定义日志 - 任务参数:&{2 TestJob1  SERIAL_EXECUTION 0 20 1665742168008 BEAN  1665718703000 0 1}
自定义日志 - 任务[2]开始执行:TestJob1
test one taskTestJob1 param： log_id:20
自定义日志 - 任务回调成功:{"code":200,"msg":null,"content":null}
自定义日志 - 任务参数:&{2 TestJob1  SERIAL_EXECUTION 0 21 1665742169009 BEAN  1665718703000 0 1}
自定义日志 - 任务[2]开始执行:TestJob1
test one taskTestJob1 param： log_id:21
自定义日志 - 任务回调成功:{"code":200,"msg":null,"content":null}
自定义日志 - 任务参数:&{2 TestJob1  SERIAL_EXECUTION 0 22 1665742170008 BEAN  1665718703000 0 1}
自定义日志 - 任务[2]开始执行:TestJob1
test one taskTestJob1 param： log_id:22
自定义日志 - 任务回调成功:{"code":200,"msg":null,"content":null}
自定义日志 - 任务参数:&{2 TestJob1  SERIAL_EXECUTION 0 23 1665742171009 BEAN  1665718703000 0 1}
自定义日志 - 任务[2]开始执行:TestJob1
test one taskTestJob1 param： log_id:23
自定义日志 - 任务回调成功:{"code":200,"msg":null,"content":null}
自定义日志 - 任务参数:&{2 TestJob1  SERIAL_EXECUTION 0 24 1665742172012 BEAN  1665718703000 0 1}
自定义日志 - 任务[2]开始执行:TestJob1
test one taskTestJob1 param： log_id:24
自定义日志 - 任务回调成功:{"code":200,"msg":null,"content":null}
自定义日志 - 任务参数:&{2 TestJob1  SERIAL_EXECUTION 0 25 1665742173005 BEAN  1665718703000 0 1}
自定义日志 - 任务[2]开始执行:TestJob1
test one taskTestJob1 param： log_id:25
```

OK 木有问题，我们成功注册并启动了这个任务

我们看下调度日志：

[![image-20221014181145291](https://marksuper.xyz/picture/file/xxl-job/xxl-job-6.png)](https://marksuper.xyz/picture/file/xxl-job/xxl-job-6.png)

点击日志的执行备注的查看，发现我们这个任务函数 return 的 test done 记录在了执行备注中。如果任务运行失败我们就可以把错误的字符串 return 出去他就会记录在执行备注中。

### 注册一个可以手动终止的任务

如果这个任务要循环处理多个数据并且耗时很长，我们需要在其执行到我们需要的效果后手动中断这样的需求可以满足吗，是可以的我们看我作者的第二个例子然后注册任务2

```go
func TestJob2(ctx context.Context, param *xxl.RunReq) (msg string) {
 num := 1
 for {

 select {
 case <-ctx.Done():
 fmt.Println("task" + param.ExecutorHandler + "被手动终止")
 return
 default:
 num++
 time.Sleep(10 * time.Second)
 fmt.Println("test one task"+param.ExecutorHandler+" param："+param.ExecutorParams+"执行行", num)
 if num > 10 {
 fmt.Println("test one task" + param.ExecutorHandler + " param：" + param.ExecutorParams + "执行完毕！")
 return
 }
 }
 }

}
```

在这个例子中我们不断的循环执行一个打印任务，然后以 ctx.Done 接收到信号为标志来终止任务，其实看到这个任务我们就该能想到在x xl-job-admin 平台去手动终止一个任务，xxl-job 一定是触发了执器的一个接口这个接口一定是触发了任务的 ctx 的 cancel 方法，为了验证我们的猜想我们去看下刚注册的/kill 路由对应的 e.killTask 方法：

```go
//删除一个任务
func (e *executor) killTask(writer http.ResponseWriter, request *http.Request) {
 e.mu.Lock()
 defer e.mu.Unlock()
 req, _ := ioutil.ReadAll(request.Body)
 param := &killReq{}
 _ = json.Unmarshal(req, &param)
 if !e.runList.Exists(Int64ToStr(param.JobID)) {
 _, _ = writer.Write(returnKill(param, FailureCode))
 e.log.Error("任务[" + Int64ToStr(param.JobID) + "]没有运行")
 return
 }
 task := e.runList.Get(Int64ToStr(param.JobID))
 task.Cancel()
 e.runList.Del(Int64ToStr(param.JobID))
 _, _ = writer.Write(returnGeneral())
}
```

果然他调用了task.Cancel。我们又猜对了。

接下来我把这个任务注册为 TestJob2 然后重启执行器服务。

```go

func StartXXLJobClient(conf *conf.DemoConfig) {

 exec := xxl.NewExecutor(
 xxl.RegistryKey(conf.XXLJobConf.AppName), //执行器名称
 xxl.ServerAddr(conf.XXLJobConf.AdminAddress),
 xxl.AccessToken(conf.XXLJobConf.Token),                     //请求令牌(默认为空)
 xxl.ExecutorPort(strconv.Itoa(conf.XXLJobConf.ClientPort)), //默认9999（非必填）
 xxl.SetLogger(&logger{}),                                   //自定义日志
 //xxl.ExecutorIp("127.0.0.1"),      //可自动获取
 )
 exec.Init()
 //设置日志查看handler
 exec.LogHandler(func(req *xxl.LogReq) *xxl.LogRes {
 return &xxl.LogRes{Code: 200, Msg: "", Content: xxl.LogResContent{
 FromLineNum: req.FromLineNum,
 ToLineNum:   2,
 LogContent:  "这个是自定义日志handler",
 IsEnd:       true,
 }}
 })
 //注册任务handler
 exec.RegTask("TestJob1", job.TestJob1)
 exec.RegTask("TestJob2", job.TestJob2)
 log.Fatal(exec.Run())
}
```

然后和刚刚一样，在 xxl-job-admin 中的 -> 任务管理界面 -> 新增

[![image-20221014182537985](https://marksuper.xyz/picture/file/xxl-job/xxl-job-8.png)](https://marksuper.xyz/picture/file/xxl-job/xxl-job-8.png)

这个我就不想多说了，看图就好。

注册成功后我们点击执行一次，执行参数还是abc。看下执行器日志：

    自定义日志 - 任务参数:&{3 TestJob2 abc SERIAL_EXECUTION 0 121 1665743199939 BEAN  1665743158000 0 1}
    自定义日志 - 任务[3]开始执行:TestJob2
    自定义日志 - 执行器注册成功:{"code":200,"msg":null,"content":null}
    test one taskTestJob2 param：abc执行行 2
    test one taskTestJob2 param：abc执行行 3
    自定义日志 - 执行器注册成功:{"code":200,"msg":null,"content":null}
    test one taskTestJob2 param：abc执行行 4
    test one taskTestJob2 param：abc执行行 5
    自定义日志 - 执行器注册成功:{"code":200,"msg":null,"content":null}
    test one taskTestJob2 param：abc执行行 6

可以看到他在运行中

我们在看下调度日志：

[![image-20221014182837379](https://marksuper.xyz/picture/file/xxl-job/xxl-job-9.png)](https://marksuper.xyz/picture/file/xxl-job/xxl-job-9.png)

我们在该日志下的操作列表中选择终止任务，可以看到日志：

```shell

自定义日志 - 执行器注册成功:{"code":200,"msg":null,"content":null}
test one taskTestJob2 param：abc执行行 5
test one taskTestJob2 param：abc执行行 6
自定义日志 - 执行器注册成功:{"code":200,"msg":null,"content":null}
test one taskTestJob2 param：abc执行行 7
taskTestJob2被手动终止
```

然后我们再看下调度日志：

[![image-20221014183213370](https://marksuper.xyz/picture/file/xxl-job/xxl-job-10.png)](https://marksuper.xyz/picture/file/xxl-job/xxl-job-10.png)

OK 成功！！

## 总结

希望通过我的讲解大家都可以在你们的实战场景中成功使用 xxl-job，就算你们用不到你也可以从这个 xxl-job 的设计思路中进行总结和汲取，希望该文对你有帮助，有问题欢迎留言。
