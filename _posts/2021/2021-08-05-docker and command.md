---
layout: post
title: Docker 可能会用到的命令
category: docker
tags: [docker]
keywords: docker,Docker 可能会用到的命令
excerpt: Docker 可能会用到的命令。
---

# Docker 可能会用到的命令

## 一、容器生命周期管理

## `1、start/stop/restart`

`docker start` :**启动**一个或多个已经被停止的容器
`docker stop`:**停止**一个运行中的容器
`docker restart` :**重启**容器

```
语法：
```



```css
docker start [OPTIONS] CONTAINER [CONTAINER...]
docker stop [OPTIONS] CONTAINER [CONTAINER...]
docker restart [OPTIONS] CONTAINER [CONTAINER...]
实例
```



```undefined
启动已被停止的容器myrunoob
docker start myrunoob

停止运行中的容器myrunoob
docker stop myrunoob

重启容器myrunoob
docker restart myrunoob
```

## `2、rm`



```undefined
docker rm ：删除一个或多个容器。
语法
```



```css
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

## `OPTIONS说明：`



```go
-f :`通过 SIGKILL 信号强制删除一个运行中的容器。`

-l :`移除容器间的网络连接，而非容器本身。`

-v :`删除与容器关联的卷。`
实例
```



```jsx
强制删除容器 db01、db02：

docker rm -f db01 db02
移除容器 nginx01 对容器 db01 的连接，连接名 db：

docker rm -l db 
删除容器 nginx01, 并删除容器挂载的数据卷：

docker rm -v nginx01

删除所有已经停止的容器：
docker rm $(docker ps -a -q)
```

# `3、pause/unpause`

`docker pause` :**暂停**容器中**所有**的进程。

`docker unpause` :**恢复**容器中**所有**的进程。

```
语法
```



```css
docker pause CONTAINER [CONTAINER...]
docker unpause CONTAINER [CONTAINER...]
实例
```



```undefined
暂停数据库容器db01提供服务。
docker pause db01

恢复数据库容器 db01 提供服务。
docker unpause db01
```

# `4、create`

```
docker create` ：创建一个新的容器但不启动它 (用法同run)
`语法
```



```css
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
实例`
使用docker镜像`nginx:latest`创建一个容器,并将容器命名为`myrunoob
```



```ruby
runoob@runoob:~$ docker create  --name myrunoob  nginx:latest      
09b93464c2f75b7b69f83d56a9cfc23ceb50a48a9db7652ee4c27e3e2cb1961f
```

# `5、kill`

`docker kill` :杀掉一个运行中的容器。

```
语法
```



```css
docker kill [OPTIONS] CONTAINER [CONTAINER...]
OPTIONS说明：
```



```undefined
-s :向容器发送一个信号
```

`实例`
杀掉运行中的容器mynginx



```ruby
runoob@runoob:~$ docker kill -s KILL mynginx
mynginx
```

## 二、容器操作

# `1、inspect`



```undefined
docker inspect : 获取容器/镜像的元数据。
语法
```



```ruby
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
OPTIONS说明：
```



```tsx
-f :指定返回值的模板文件。

-s :显示总的文件大小。

--type :为指定类型返回JSON。
```

`实例`
获取镜像mysql:5.6的元信息。



```ruby
runoob@runoob:~$ docker inspect mysql:5.6
[
    {
        "Id": "sha256:2c0964ec182ae9a045f866bbc2553087f6e42bfc16074a74fb820af235f070ec",
        "RepoTags": [
            "mysql:5.6"
        ],
        "RepoDigests": [],
        "Parent": "",
        "Comment": "",
        "Created": "2016-05-24T04:01:41.168371815Z",
        "Container": "e0924bc460ff97787f34610115e9363e6363b30b8efa406e28eb495ab199ca54",
        "ContainerConfig": {
            "Hostname": "b0cf605c7757",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "3306/tcp": {}
            },
```

获取正在运行的容器mymysql的 IP。



```ruby
runoob@runoob:~$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mymysql
172.17.0.3
```

# `2、top`

`docker top` :查看容器中运行的进程信息，支持 ps 命令参数。

```
语法
```



```css
docker top [OPTIONS] CONTAINER [ps OPTIONS]
```

容器运行时不一定有/bin/bash终端来交互执行top命令，而且容器还不一定有top命令，可以使用docker top来实现查看container中正在运行的进程。

`实例`
查看容器mymysql的进程信息。



```ruby
runoob@runoob:~/mysql$ docker top mymysql
UID    PID    PPID    C      STIME   TTY  TIME       CMD
999    40347  40331   18     00:58   ?    00:00:02   mysqld
```

查看所有运行容器的进程信息。



```bash
for i in  `docker ps |grep Up|awk '{print $1}'`;do echo \ &&docker top $i; done
```

# `3、attach`

`docker attach` :连接到正在运行中的容器。

```
语法
```



```css
docker attach [OPTIONS] CONTAINER
```

要attach上去的容器必须正在运行，可以同时连接上同一个container来共享屏幕（与screen命令的attach类似）。

官方文档中说attach后可以通过CTRL-C来detach，但实际上经过我的测试，如果container当前在运行bash，CTRL-C自然是当前行的输入，没有退出；如果container当前正在前台运行进程，如输出nginx的access.log日志，CTRL-C不仅会导致退出容器，而且还stop了。这不是我们想要的，detach的意思按理应该是脱离容器终端，但容器依然运行。好在attach是可以带上--sig-proxy=false来确保CTRL-D或CTRL-C不会关闭容器。

`实例`
容器mynginx将访问日志指到标准输出，连接到容器查看访问信息。



```ruby
runoob@runoob:~$ docker attach --sig-proxy=false mynginx 192.168.239.1 - - [10/Jul/2016:16:54:26 +0000] "GET / HTTP/1.1" 304 0 "-" 
"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.93 Safari/537.36" "-"
```

# `4、events`

`docker events` : 从服务器获取实时事件

```
语法
```



```css
docker events [OPTIONS]
OPTIONS说明：
```



```ruby
-f ：根据条件过滤事件；
--since ：从指定的时间戳后显示所有事件;
--until ：流水时间显示到指定的时间为止；
```

`实例`
显示docker 2016年7月1日后的所有事件。



```ruby
runoob@runoob:~/mysql$ docker events  --since="1467302400"
2016-07-08T19:44:54.501277677+08:00 network connect 66f958fd13dc4314ad20034e576d5c5eba72e0849dcc38ad9e8436314a4149d4 (container=b8573233d675705df8c89796a2c2687cd8e36e03646457a15fb51022db440e64, name=bridge, type=bridge)
2016-07-08T19:44:54.723876221+08:00 container start b8573233d675705df8c89796a2c2687cd8e36e03646457a15fb51022db440e64 (image=nginx:latest, name=elegant_albattani)
2016-07-08T19:44:54.726110498+08:00 container resize b8573233d675705df8c89796a2c2687cd8e36e03646457a15fb51022db440e64 (height=39, image=nginx:latest, name=elegant_albattani, width=167)
2016-07-08T19:46:22.137250899+08:00 container die b8573233d675705df8c89796a2c2687cd8e36e03646457a15fb51022db440e64 (exitCode=0, image=nginx:latest, name=elegant_albattani)
```

显示docker 镜像为mysql:5.6 2016年7月1日后的相关事件。



```ruby
runoob@runoob:~/mysql$ docker events -f "image"="mysql:5.6" --since="1467302400" 
2016-07-11T00:38:53.975174837+08:00 container start 96f7f14e99ab9d2f60943a50be23035eda1623782cc5f930411bbea407a2bb10 (image=mysql:5.6, name=mymysql)
2016-07-11T00:51:17.022572452+08:00 container kill 96f7f14e99ab9d2f60943a50be23035eda1623782cc5f930411bbea407a2bb10 (image=mysql:5.6, name=mymysql, signal=9)
2016-07-11T00:51:17.132532080+08:00 container die 96f7f14e99ab9d2f60943a50be23035eda1623782cc5f930411bbea407a2bb10 (exitCode=137, image=mysql:5.6, name=mymysql)
2016-07-11T00:51:17.514661357+08:00 container destroy 96f7f14e99ab9d2f60943a50be23035eda1623782cc5f930411bbea407a2bb10 (image=mysql:5.6, name=mymysql)
2016-07-11T00:57:18.551984549+08:00 container create c8f0a32f12f5ec061d286af0b1285601a3e33a90a08ff1706de619ac823c345c (image=mysql:5.6, name=mymysql)
2016-07-11T00:57:18.557405864+08:00 container attach c8f0a32f12f5ec061d286af0b1285601a3e33a90a08ff1706de619ac823c345c (image=mysql:5.6, name=mymysql)
2016-07-11T00:57:18.844134112+08:00 container start c8f0a32f12f5ec061d286af0b1285601a3e33a90a08ff1706de619ac823c345c (image=mysql:5.6, name=mymysql)
2016-07-11T00:57:19.140141428+08:00 container die c8f0a32f12f5ec061d286af0b1285601a3e33a90a08ff1706de619ac823c345c (exitCode=1, image=mysql:5.6, name=mymysql)
2016-07-11T00:58:05.941019136+08:00 container destroy c8f0a32f12f5ec061d286af0b1285601a3e33a90a08ff1706de619ac823c345c (image=mysql:5.6, name=mymysql)
2016-07-11T00:58:07.965128417+08:00 container create a404c6c174a21c52f199cfce476e041074ab020453c7df2a13a7869b48f2f37e (image=mysql:5.6, name=mymysql)
2016-07-11T00:58:08.188734598+08:00 container start a404c6c174a21c52f199cfce476e041074ab020453c7df2a13a7869b48f2f37e (image=mysql:5.6, name=mymysql)
2016-07-11T00:58:20.010876777+08:00 container top a404c6c174a21c52f199cfce476e041074ab020453c7df2a13a7869b48f2f37e (image=mysql:5.6, name=mymysql)
2016-07-11T01:06:01.395365098+08:00 container top a404c6c174a21c52f199cfce476e041074ab020453c7df2a13a7869b48f2f37e (image=mysql:5.6, name=mymysql)
```

如果指定的时间是到秒级的，需要将时间转成时间戳。如果时间为日期的话，可以直接使用，如--since="2016-07-01"。

# `5、wait`

`docker wait` : 阻塞运行直到容器停止，然后打印出它的退出代码。

```
语法
```



```css
docker wait [OPTIONS] CONTAINER [CONTAINER...]
实例
```



```bash
docker wait CONTAINER
```

## 三、容器rootfs命令

# `1、commit`

`docker commit` :从容器创建一个新的镜像。

```
语法
```



```css
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
OPTIONS说明：
```



```swift
-a :提交的镜像作者；
-c :使用Dockerfile指令来创建镜像；
-m :提交时的说明文字；
-p :在commit时，将容器暂停。
```

`实例`
将容器a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。



```ruby
runoob@runoob:~$ docker commit -a "runoob.com" -m "my apache" a404c6c174a2  mymysql:v1 
sha256:37af1236adef1544e8886be23010b66577647a40bc02c0885a6600b33ee28057
runoob@runoob:~$ docker images mymysql:v1
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mymysql             v1                  37af1236adef        15 seconds ago      329 MB
```

# `2、 cp`

`cocker cp` :用于容器与主机之间的数据拷贝。

```
语法
```



```ruby
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
OPTIONS说明：
```



```undefined
-L :保持源目标中的链接
实例
```



```undefined
将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下。
docker cp /www/runoob 96f7f14e99ab:/www/

将主机/www/runoob目录拷贝到容器96f7f14e99ab中，目录重命名为www。
docker cp /www/runoob 96f7f14e99ab:/www

将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。
docker cp  96f7f14e99ab:/www /tmp/
```

# `3、diff`

`docker diff` : 检查容器里文件结构的更改。

```
语法
```



```css
docker diff [OPTIONS] CONTAINER
```

`实例`
查看容器mymysql的文件结构更改。



```ruby
runoob@runoob:~$ docker diff mymysql
A /logs
A /mysql_data
C /run
C /run/mysqld
A /run/mysqld/mysqld.pid
A /run/mysqld/mysqld.sock
C /tmp
```

## 四、镜像仓库

# `1、login/logout`

`docker login` : 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

`docker logout` : 登出一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

```
语法
```



```css
docker login [OPTIONS] [SERVER]
docker logout [OPTIONS] [SERVER]
OPTIONS说明：
```



```undefined
-u :登陆的用户名

-p :登陆的密码
实例
```



```bash
登陆到Docker Hub
docker login -u 用户名 -p 密码

登出Docker Hub
docker logout
```

# `2、pull`

`docker pull`: 从镜像仓库中拉取或者更新指定镜像

```
语法
```



```ruby
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
OPTIONS说明：
```



```bash
-a :拉取所有 tagged 镜像

--disable-content-trust :忽略镜像的校验,默认开启
实例
```



```undefined
从Docker Hub下载java最新版镜像。
docker pull java

从Docker Hub下载REPOSITORY为java的所有镜像。
docker pull -a java
```

# `3、push`

```
docker push` : 将本地的镜像上传到镜像仓库,要先登陆到镜像仓库
`语法
```



```css
docker push [OPTIONS] NAME[:TAG]
OPTIONS说明：
```



```bash
--disable-content-trust :忽略镜像的校验,默认开启
```

`实例`
上传本地镜像myapache:v1到镜像仓库中。



```css
docker push myapache:v1
```

# `4、search`

```
docker search` : 从Docker Hub查找镜像
`语法
```



```css
docker search [OPTIONS] TERM
OPTIONS说明：
```



```xml
--automated :只列出 automated build类型的镜像；

--no-trunc :显示完整的镜像描述；

-f <过滤条件>:列出收藏数不小于指定值的镜像。
```

`实例`
从 Docker Hub 查找所有镜像名包含 java，并且收藏数大于 10 的镜像



```kotlin
runoob@runoob:~$ docker search -f stars=10 java
NAME                  DESCRIPTION                           STARS   OFFICIAL   AUTOMATED
java                  Java is a concurrent, class-based...   1037    [OK]       
anapsix/alpine-java   Oracle Java 8 (and 7) with GLIBC ...   115                [OK]
develar/java                                                 46                 [OK]
isuper/java-oracle    This repository contains all java...   38                 [OK]
lwieske/java-8        Oracle Java 8 Container - Full + ...   27                 [OK]
nimmis/java-centos    This is docker images of CentOS 7...   13                 [OK]
参数说明：
```



```go
`NAME`: 镜像仓库源的名称

`DESCRIPTION`: 镜像的描述

`OFFICIAL`: 是否 docker 官方发布

`stars`: 类似 Github 里面的 star，表示点赞、喜欢的意思。

`AUTOMATED`: 自动构建。
```

## 五、本地镜像管理

# `1、rmi`

`docker rmi`: 删除本地一个或多少镜像。

```
语法
```



```css
docker rmi [OPTIONS] IMAGE [IMAGE...]
OPTIONS说明：
```



```undefined
-f :强制删除；

--no-prune :不移除该镜像的过程镜像，默认移除；
实例
```

强制删除本地镜像 runoob/ubuntu:v4。



```ruby
root@runoob:~# docker rmi -f runoob/ubuntu:v4
Untagged: runoob/ubuntu:v4
Deleted: sha256:1c06aa18edee44230f93a90a7d88139235de12cd4c089d41eed8419b503072be
Deleted: sha256:85feb446e89a28d58ee7d80ea5ce367eebb7cec70f0ec18aa4faa874cbd97c73
```

# `2、tag`

`docker tag` : 标记本地镜像，将其归入某一仓库。

```
语法
```



```css
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
实例
```

将镜像ubuntu:15.10标记为 runoob/ubuntu:v3 镜像。



```ruby
root@runoob:~# docker tag ubuntu:15.10 runoob/ubuntu:v3
root@runoob:~# docker images   runoob/ubuntu:v3
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
runoob/ubuntu       v3                  4e3b13c8a266        3 months ago        136.3 MB
```

# `3、build`

`docker build` 命令用于使用 Dockerfile 创建镜像。

```
语法
```



```ruby
docker build [OPTIONS] PATH | URL | -
OPTIONS说明：
```



```objectivec
--build-arg=[] :设置镜像创建时的变量；

--cpu-shares :设置 cpu 使用权重；

--cpu-period :限制 CPU CFS周期；

--cpu-quota :限制 CPU CFS配额；

--cpuset-cpus :指定使用的CPU id；

--cpuset-mems :指定使用的内存 id；

--disable-content-trust :忽略校验，默认开启；

-f :指定要使用的Dockerfile路径；

--force-rm :设置镜像过程中删除中间容器；

--isolation :使用容器隔离技术；

--label=[] :设置镜像使用的元数据；

-m :设置内存最大值；

--memory-swap :设置Swap的最大值为内存+swap，"-1"表示不限swap；

--no-cache :创建镜像的过程不使用缓存；

--pull :尝试去更新镜像的新版本；

--quiet, -q :安静模式，成功后只输出镜像 ID；

--rm :设置镜像成功后删除中间容器；

--shm-size :设置/dev/shm的大小，默认值是64M；

--ulimit :Ulimit配置。

--squash :将 Dockerfile 中所有的操作压缩为一层。

--tag, -t: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。

--network: 默认 default。在构建期间设置RUN指令的网络模式
实例
```



```jsx
使用当前目录的 Dockerfile 创建镜像，标签为 runoob/ubuntu:v1。
docker build -t runoob/ubuntu:v1 . 

使用URL github.com/creack/docker-firefox 的 Dockerfile 创建镜像。
docker build github.com/creack/docker-firefox

也可以通过 -f Dockerfile 文件的位置：
$ docker build -f /path/to/a/Dockerfile .

在 Docker 守护进程执行 Dockerfile 中的指令前，首先会对 Dockerfile 进行语法检查，有语法错误时会返回：
$ docker build -t test/myapp .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
```

# `4、history`

`docker history :`查看指定镜像的创建历史。

```
语法
```



```bash
docker history [OPTIONS] IMAGE
OPTIONS说明：
```



```bash
-H :以可读的格式打印镜像大小和日期，默认为true；

--no-trunc :显示完整的提交记录；

-q :仅列出提交记录ID。
```

`实例`
查看本地镜像runoob/ubuntu:v3的创建历史。



```ruby
root@runoob:~# docker history runoob/ubuntu:v3
IMAGE             CREATED           CREATED BY                                      SIZE      COMMENT
4e3b13c8a266      3 months ago      /bin/sh -c #(nop) CMD ["/bin/bash"]             0 B                 
<missing>         3 months ago      /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$/   1.863 kB            
<missing>         3 months ago      /bin/sh -c set -xe   && echo '#!/bin/sh' > /u   701 B               
<missing>         3 months ago      /bin/sh -c #(nop) ADD file:43cb048516c6b80f22   136.3 MB
```

# `5、save`

`docker save :`将指定镜像保存成 tar 归档文件。

```
语法
```



```css
docker save [OPTIONS] IMAGE [IMAGE...]
OPTIONS 说明：
```



```undefined
-o :输出到的文件。
```

`实例`
将镜像 runoob/ubuntu:v3 生成 my_ubuntu_v3.tar 文档



```ruby
runoob@runoob:~$ docker save -o my_ubuntu_v3.tar runoob/ubuntu:v3
runoob@runoob:~$ ll my_ubuntu_v3.tar
-rw------- 1 runoob runoob 142102016 Jul 11 01:37 my_ubuntu_v3.ta
```

# `6、load`

`docker load :`导入使用 `docker save`命令导出的镜像。

```
语法
```



```css
docker load [OPTIONS]
OPTIONS 说明：
```



```undefined
--input , -i : **指定导入的文件，代替 STDIN。

--quiet , -q : **精简输出信息。
实例
```

导入镜像：



```csharp
<pre class="prettyprint prettyprinted" style="border-width: 1px 1px 1px 4px; border-style: solid; border-color: rgb(221, 221, 221); border-image: initial; margin: 15px auto; padding: 10px 15px; font: 400 12px / 20px Menlo, Monaco, Consolas, &quot;Andale Mono&quot;, &quot;lucida console&quot;, &quot;Courier New&quot;, monospace; white-space: pre-wrap; word-break: break-all; overflow-wrap: break-word; background: url(&quot;/images/codecolorer_bg.gif&quot;) center top rgb(251, 251, 251); color: rgb(51, 51, 51); letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;">$ docker image ls

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

$ docker load < busybox.tar.gz Loaded image: busybox:latest
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest 769b9341d937  7 weeks ago 2.489 MB

$ docker load --input fedora.tar Loaded image: fedora:rawhide Loaded image: fedora:20 $ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest 769b9341d937  7 weeks ago 2.489 MB
fedora              rawhide 0d20aec6529d  7 weeks ago 387 MB
fedora 20  58394af37342  7 weeks ago 385.5 MB
fedora              heisenbug 58394af37342  7 weeks ago 385.5 MB
fedora              latest 58394af37342  7 weeks ago 385.5 MB</pre>
```

## 六、info|version

# `1、info`

`docker info` : 显示 Docker 系统信息，包括镜像和容器数。。

```
语法
```



```css
docker info [OPTIONS]
实例
```

# `查看docker系统信息。`



```java
$ docker info
Containers: 12
Images: 41
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 66
 Dirperm1 Supported: false
Execution Driver: native-0.2
Logging Driver: json-file
Kernel Version: 3.13.0-32-generic
Operating System: Ubuntu 14.04.1 LTS
CPUs: 1
Total Memory: 1.954 GiB
Name: iZ23mtq8bs1Z
ID: M5N4:K6WN:PUNC:73ZN:AONJ:AUHL:KSYH:2JPI:CH3K:O4MK:6OCX:5OYW
```

# `2、version`

`docker version` :显示 Docker 版本信息。

```
语法
```



```css
docker version [OPTIONS]
OPTIONS说明：
```

-f :指定返回值的模板文件。

`实例`
显示 Docker 版本信息。



```ruby
$ docker version
Client:
 Version:      1.8.2
 API version:  1.20
 Go version:   go1.4.2
 Git commit:   0a8c2e3
 Built:        Thu Sep 10 19:19:00 UTC 2015
 OS/Arch:      linux/amd64

Server:
 Version:      1.8.2
 API version:  1.20
 Go version:   go1.4.2
 Git commit:   0a8c2e3
 Built:        Thu Sep 10 19:19:00 UTC 2015
 OS/Arch:      linux/amd64
```