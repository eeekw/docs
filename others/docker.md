# Docker

## Dockerfile

### 解析器指令

`# directive=value`

* `syntax`
  * The syntax directive defines the location of the Dockerfile builder that is used for building the current Dockerfile.
  * `# syntax=docker/dockerfile:1.0.0-experimental`
* `escape`
  * The escape directive sets the character used to escape characters in a Dockerfile. If not specified, the default escape character is .
  * 转义字符：The escape character is used both to escape characters in a line, and to escape a newline.
  * `# escape=\ (backslash)` or `# escape=` \(backtick\)\`
  * 若未指定，默认转义字符是`\`

### .dockerignore

* 排除`ADD`、`COPY`中匹配的文件和目录

### 环境变量

### From

`FROM [--platform=<platform>] <image> [AS <name>]`

`FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]`

`FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]`

初始化一个新的构建阶段并设置基础镜像

### ARG

`ARG`唯一可以在`From`之前的指令

### RUN

`RUN <command>`//shell 格式

`RUN ["executable", "param1", "param2"]`//exec 格式

`RUN`指令将在当前镜像顶部的新层上执行命令并提交这个结果

### LABEL

`LABEL <key>=<value> <key>=<value> <key>=<value> ...`

`LABEL`指令给镜像添加元数据

减小最终镜像的大小

```text
LABEL multi.label1="value1" multi.label2="value2" other="value3"
```

```text
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

镜像 继承基础镜像或父镜像的Labels

`docker inspect` 查看镜像labels

### MAINTAINER

已废弃

设置作者字段，使用`LABEL`指令替代

`LABEL maintainer="SvenDowideit@home.org.au"`

### EXPOSE

`EXPOSE <port> [<port>/<protocol>...]`

通知Docker该容器运行时监听指定的网络端口。可以指定在TCP还是UDP协议上监听，默认TCP协议。

`EXPOSE`指令并不会发布端口，只是说明哪个端口打算被发布。想要实际发布端口，通过在`docker run`上`-p`标志来发布并映射一个或多个端口，`-P`标志发布所有EXPOSE的端口并映射到高阶端口。

`EXPOSE 80/udp`

同时`EXPOSE` TCP和UDP

```text
EXPOSE 80/tcp
EXPOSE 80/udp
```

`docker network`命令支持创建在容器之间通信的网络，不需要发布指定端口

### ENV

```text
ENV <key> <value>
ENV <key>=<value> ...
```

设置环境变量

`<value>`将在构建阶段中所有后续指令的环境中使用

`ENV <key> <value>`在第一个空格后的所有字符作为`<value>`包括空格

```text
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy
```

`ENV <key>=<value> ...`允许同时设置多个变量

```text
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
```

容器运行时ENV设置的环境变量将被保留，通过`docker inspect`查看值，通过`docker run —env <key>=<value>`改变

### ADD

`ADD [--chown=<user>:<group>] <src>... <dest>`

`ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]`有空格的路径需要此格式

`--chown`只被支持用于构建Linux容器

`ADD`指令拷贝来自`<src>`的文件、目录或远程文件URL并添加到镜像的文件系统的`<dest>`路径

可以指定多个`<src>`，如果它们是文件或目录，则它们的路径相对于构建上下文解析

可以包含通配符

```text
ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
```

`<dest>`是绝对路径，或是相对于`WORKDIR`的路径

```text
ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
```

添加包含特殊字符（例如`[`和`]`）的文件或目录时，您需要按照Golang规则转义那些路径，以防止将它们视为匹配模式

```text
ADD arr[[]0].txt /mydir/    # copy a file named "arr[0].txt" to /mydir/
```

在`<src>`是远程`URL`的情况下，添加后的文件权限为600

`ADD`指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢

`--chown=<user>:<group>`改变文件的所属用户及所属组

```text
ADD --chown=55:mygroup files* /somedir/
ADD --chown=bin files* /somedir/
ADD --chown=1 files* /somedir/
ADD --chown=10:11 files* /somedir/
```

如果通过`STDIN`传入`Dockerfile`来构建`(docker build - < somefile)`，便没有构建上下文，`ADD`的只能是URL。如果通过`STDIN`传入压缩包`(docker build - < archive.tar.gz)`，压缩包根目录下的Dockerfile和其余部分将用作构建上下文

规则

* `<src>`路径必须在构建上下文内；不能`COPY ../something /something`，因为`docker build`的第一步是将上下文目录发送到docker daemon\(守护程序\)
* 如果`<src>`目录，目录下的所有内容都会被复制，包括文件的元数据（目录本身不会被复制，只有它的内容）
* 如果`<src>`是任何类型的文件，则会将其及其元数据一起复制。如果`<dest>`以`/`结束，它将被认为是一个目录，的内容将被写入到`<dest>/<src>`
* 如果直接或使用通配符指定了多个`<src>`，`<dest>`必须是目录且必须以`/`结束
* 如果`<dest>`没有以`/`结束，它将被视为常规文件，`<src>`里的内容将被写入`<dest>`
* 如果`<dest>`不存在，路径下所有不存在的路径都会被创建
* 如果`<src>`是一个`URL`，没有以`/`结束，文件被下载并复制到`<dest>`中
* 如果`<src>`是一个`URL`，`<dest>`以`/`结束，文件被下载到`<dest>/<filename>`，文件名从`URL`推断出
* 如果 `<src>` 为一个 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，然后被解压为目录。来自URLs的压缩包不会被解压。当一个目录被复制或解压，行为与`tar -x`相同，会和`<dest>`路径下的内容合并

### COPY

`COPY [--chown=<user>:<group>] <src>... <dest>`

`COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]` 有空格的路径需要此格式

同`ADD`指令

与`ADD`区别

* 不能复制远程文件URL

推荐使用`COPY`

### CMD

`CMD ["executable","param1","param2"]`//exec格式

`CMD ["param1","param2"]`//作为`ENTRYPOINT`默认参数

`CMD command param1 param2`//shell 格式

Dockerfile中只能有一个`CMD`指令

`CMD`主要目的是给容器在运行时指定默认的执行命令。默认的命令包含一个可执行程序，在指定`ENTRYPOINT`指令的情况下，`CMD`用于给`ENTRYPOINT`提供参数。

> 当`CMD`用于给`ENTRYPOINT`提供默认参数，`CMD`和`ENTRYPOINT`需要使用JSON数组格式来指定
>
> _exec_ form 被解析为JSON数组，意味着必须使用双引号而不是单引号
>
> 与_shell_ form不同，_exec_ form不会调用命令_shell_。

如果`CMD`使用_shell_ form，`<command>`将在`/bin/sh -c`中执行

```text
FROM ubuntu
CMD echo "This is a test." | wc -
```

如果不想在_shell_中运行`<command>`，必须使用_exec_ form

```text
FROM ubuntu
CMD ["/usr/bin/wc","--help"]
```

> 如果想每次容器运行相同的可执行文件，应该考虑使用`ENTRYPOINT`，如果`docker run`指定了参数，将覆盖`CMD`指定的默认值

### ENTRYPOINT

`ENTRYPOINT ["executable", "param1", "param2"]` exec格式；推荐

`ENTRYPOINT command param1 param2`shell格式

`ENTRYPOINT`允许您配置容器作为可执行文件运行。

`docker run -i -t --rm -p 80:80 nginx`

`docker run <image>`的命令行参数将被附加到`ENTRYPOINT` _exec_ form的所有元素之后并覆盖`CMD`指定的所有元素

_shell_ form的`ENTRYPOINT` 将作为`/bin/sh -c`的子命令启动，该子命令不传递信号，意味着可执行程序将不是容器的`PID 1`且不会接收Unix信号，因此你的可执行程序将不会从`docker stop <container>`接收`SIGTERM`信号

为了确保`docker stop`正确的发送信号给`ENTRYPOINT`执行程序，需要用`exec`启动

```text
FROM ubuntu
ENTRYPOINT exec top -b
```

### VOLUME

`VOLUME ["/data"]`

`VOLUME /var/log or VOLUME /var/log /var/db`//可指定多个参数

`VOLUMN`指令创建一个指定名字的挂载点并绑定到来自主机或其它容器的外部的卷中。

[更多](https://docs.docker.com/engine/tutorials/dockervolumes/)

`docker run`命令使用基础镜像内指定位置上任何数据来初始新创建的卷

* 基于windows的容器，容器内的volumn的目标必须是
  * 一个不存在或空的目录
  * C以外的驱动器
* 在**Dockerfile内，任何在声明了卷后更改的卷内的数据，都将被丢弃。**
* `VOLUME ["/data"]`被解析为JSON数组，意味着必须使用双引号而不是单引号
* 主机目录（装载点）本质上是与主机相关的。这是为了保持映像可移植性，因为无法保证给定的主机目录在所有主机上都可用。因此，您无法从 Dockerfile 中装载主机目录。VOLUME 指令不支持指定`host-dir`参数。创建或运行容器时必须指定装载点。

```text
FROM ubuntu
RUN mkdir /myvol
RUN echo "hello world" > /myvol/greeting
VOLUME /myvol
```

### USER

```text
USER <user>[:<group>] or
USER <UID>[:<GID>]
```

`USER`指令为后面的`RUN`、`CMD`、`ENTRYPOINT`命令设置执行这些命令的用户名\(或UID\)和可选的用户组\(或GID\)

当用户没有组时，镜像将运行在`root`组

Windows下，用户必须提前创建，可以在Dockerfile中使用`net user`命令

```text
FROM microsoft/windowsservercore
    # Create Windows user in the container
    RUN net user /add patrick
    # Set it for subsequent commands
    USER patrick
```

### WORKDIR

`WORKDIR /path/to/workdir`

为后面的`RUN`、`CDM`、`ENTRYPOINT`命令设置工作目录。如果`WORKDIR`不存在，将会被创建尽管未使用

`WORKDIR`可以多次使用。如果是相对路径，将相对前面的`WORKDIR`命令指定的路径

```text
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

`pwd`的输出是`/a/b/c`

`WORKDIR`可以解析`ENV`已设置的环境变量，只能使用在`Dockerfile`中显示设置的环境变量

```text
ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```

`pwd`的输出是`/path/$DIRNAME`

### ARG

`ARG <name>[=<default value>]`

`ARG`命令定义一个变量，可以在构建时通过`docker build --build-arg <varname>=<value>`给这个变量传递值

不建议传递像github密钥，用户凭证等。使用`docker history`可以看到所有值

```text
FROM busybox
ARG user1
ARG buildno
...
```

`ARG`可以包含一个默认值

```text
FROM busybox
ARG user1=someuser
ARG buildno=1
...
```

### ONBUILD

### STOPSIGNAL

### SHELL

```text
# syntax=docker/dockerfile:1.0 

From
```

[源文档](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact)

