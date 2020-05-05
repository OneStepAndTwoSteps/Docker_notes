# Docker + Scrapy + MySQL

一、下载安装 Docker

1. centos extras repo 版本较老

2. https://download.docker.com

    1. 仓库配置文件：https://download.docker.com/linux/centos/docker-ce.repo

3. 自定义repo(清华大学开源镜像)
    
    1. wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
    
    2. 修改repo 将官网地址替换为 https://mirrors.tuna.tsinghua.edu.cn/docker-ce
    
    3. yum -y install docker-ce

4. `注意：`有可能会遇到问题 `xfs` 版本太低，需要更新。因为 `overlay2` 是建立在 `xfs` 之上的，解决方法： `yum update xfsprogs`


二、创建 Dockerfile

*   1、在 `项目的根目录` 下新建一个 `requirements.txt` 文件,将项目需要的环境依赖包都列出:

        scrapy
        pymysql


    或者指定版本号：

        scrapy>=1.4.0
        pymysql>=3.4.0


*   2、在 `项目根目录` 下新建一个 `Dockerfile` 文件，文件不加任何后缀名，修改内容如下所示：

        
        FROM python:3.6
        ENV PATH /usr/local/bin:$PATH
        ADD . /code
        WORKDIR /code
        RUN pip3 install -r requirements.txt
        CMD scrapy crawl spider_name

    第一行的 `FROM：`指令是最重的一个且必须为Dockerfile文件开篇的第一个非注释行，用于为映象文件构建过程中指定基准镜像所提供的运行环境
    
    第二行 `ENV` 是环境变量设置，将 /usr/local/bin:$PATH 赋值给 PATH，即增加 /usr/local/bin 这个环境变量路径。

    第三行 `ADD` 是将本地的代码放置到虚拟容器中。它有两个参数：第一个参数是.，代表本地当前路径；第二个参数是 /code，代表虚拟容器中的路径，也就是将本地项目所有内容放置到虚拟容器的 /code 目录下，以便于在虚拟容器中运行代码。

    第四行 `WORKDIR` 是指定工作目录，这里将刚才添加的代码路径设成工作路径。

    第五行 `RUN` 是执行某些命令来做一些环境准备工作。

    第六行 `CMD` 是容器启动命令。在容器运行时，此命令会被执行。

`注意：` 当前目录下一定要先包含 `spider` 项目，否则无法将当前目录下的文件编辑进 `docker image `，没有 `scrapy.cfg` 的支持，会报错：

    Unknown command: crawl

三、修改 mysql 连接

*   在 Docker 虚拟容器里 localhost 实际指向容器本身的运行 IP，而容器内部并没有安装 mysqld，所以爬虫无法连接 mysqld。

    *   可以使用 公网IP 连接



四、构建镜像

*   注意在镜像的后面有一个点 `.` 别忘了。

        docker build -t chinese_judg:v1 .

    `解释：` PATH指明了.，然后在当前目录下的多有文件都被发送进了Docker的守护进程，PATH指定了构建镜像时上下文查找文件的路径。

*   这样的输出就说明镜像构建成功:

        Successfully built protego PyDispatcher
        
        ................................

        Removing intermediate container 801e446f282d
        ---> 98f52afbf5b5
        Step 6/6 : CMD scrapy crawl chinese_judgment
        ---> Running in d31931604e60
        Removing intermediate container d31931604e60
        ---> f79a83d846bb
        Successfully built f79a83d846bb
        Successfully tagged chinese_judg:v1


五、运行

    docker run chinese_judg:v1


