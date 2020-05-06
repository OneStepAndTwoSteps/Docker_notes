# Docker Notes

## 删除 docker 容器

`语法：`

    docker rm CONTAINER_ID

`批量删除方法：` 想要删除 docker 镜像前，需要先删除对应的 docker 容器,。

    docker ps -a|awk '{print $1}'|xargs docker rm

## 查看/删除 docker 镜像     

查看镜像语法：

    docker images 

删除镜像语法：

    docker image rm image_id

`注意：`当存在多个相同 image ID 的镜像时，使用 image_id 的方式删除 镜像会报错：

    REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
    <none>                       <none>              cfaf8d1a99e4        18 hours ago        921MB
    chinese_judg                 v2                  2769f118de5d        22 hours ago        1GB
    chinese_judg                 v3                  2769f118de5d        22 hours ago        1GB

    执行：
    docker image rm 2769f118de5d
    
    报错：
    unable to delete 2769f118de5d (must be forced) - image is referenced in multiple repositories

解决方案：

*   使用 `REPOSITORY` 来指定删除:

        docker rmi chinese_judg:v2


## docker-compose

编写一个 `docker-compose.yml` 文件:

Compose 中有两个重要的概念：

*   `服务 (service)`：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

*   `项目 (project)`：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义。

Compose 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。


*   `build`

    指定 Dockerfile 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 Compose 将会利用它自动构建这个镜像，然后使用这个镜像。

        version: '3'
        services:
        webapp:
        build: ./dir

    你也可以使用 context 指令指定 Dockerfile 所在文件夹的路径。 使用 dockerfile 指令指定 Dockerfile 文件名。

        version: '3'
        services:
        webapp:
        build:
        context: ./dir
        dockerfile: Dockerfile-alternate


*   `command`

    覆盖容器启动后默认执行的命令。

        command: echo "hello world"

*   `dns`

    自定义 DNS 服务器。可以是一个值，也可以是一个列表。

        dns: 8.8.8.8

        dns:
        - 8.8.8.8
        - 114.114.114.114

*   `dns_search`

    配置 DNS 搜索域。可以是一个值，也可以是一个列表。

        dns_search: example.com

        dns_search:
        - domain1.example.com
        - domain2.example.com

*   `expose`

    暴露端口，但不映射到宿主机，只被连接的服务访问。

        expose:
        - "3000"
        - "8000"



*   `image`

    指定为镜像名称或镜像 ID。如果镜像在本地不存在，Compose 将会尝试拉取这个镜像。

        image: ubuntu
        image: orchardup/postgresql
        image: a4bc65fd

*   `links`

    连接其他服务的容器，可以指定服务名称。 

        web:
        links:
        -db
        -redis

*   `ports`

    暴露端口信息。 使用宿主端口：容器端口格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。
        
        ports:
        - "3000"
        - "8000:8000"
        - "49100:22"
        - "127.0.0.1:8001:8001"

*   `volumes`

    数据卷所挂载路径设置。可以设置宿主机路径 （HOST:CONTAINER） 或加上访问模式。 加载本地目录下的配置文件到容器目标地址下。

        volumes:
        - /var/lib/mysql
        - cache/:/tmp/cache
        - ~/configs:/etc/configs/:ro

*   `environment` 设置环境变量。你可以使用数组或字典两种格式。

    只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。

        environment:
        RACK_ENV: development
        SESSION_SECRET:

        environment:
        - RACK_ENV=development
        - SESSION_SECRET

*   `networks`

    配置容器连接的网络。

        version: "3"
        services:
        some-service:
        networks:
        - some-network
        - other-network
        *   depends_on

    
    解决容器的依赖、启动先后的问题。以下例子中会先启动 redis db 再启动 web

        version: '3'
        services:
        web:
        build: .
        depends_on:
        - db
        - redis
        redis:
        image: redis
        db:
        image: postgres

    注意：web 服务不会等待 redis db 「完全启动」之后才启动。

*   `container_name`

    指定容器名称。默认将会使用 项目名称_服务名称_序号 这样的格式。

        container_name: docker-web-container

    注意: 指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。

*   `restart`

    restart: always 表示如果服务启动不成功会一直尝试。

*   `working_dir`

    指定容器中工作目录。

        working_dir: /code


### docker-conpose error 解决方案：

问题：

    ERROR: Version in “./docker-compose.yml” is unsupported

解决方案：

    这种问题一般是因为

    docker-compose的版本和 ./docker-compose.yml 要求的版本对应不上

    docker-compose 的版本可以用 docker-compose --version 进行检查

    可以通过修改 ./docker-compose.yml 的version的值改好的情况




