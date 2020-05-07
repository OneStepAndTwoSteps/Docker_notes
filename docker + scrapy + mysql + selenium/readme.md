# docker + scrapy + selenium + pymysql

首先要介绍一下 [selenium/standalone-chrome](https://github.com/SeleniumHQ/docker-selenium) 

*   `selenium/standalone-chrome:` 安装有 Chrome 的 Selenium Node 节点镜像

`docker-selenium` 项目（ 镜像仓库 , 代码仓库 ）是将 selenium、webdriver、VNC server、chrome（或者firefox）集成在一个docker镜像里的项目。提供如下的功能：

*   代替原有的 remote webdriver
*   单个容器就能提供全套 selenium+webdriver+headless 浏览器的功能
*   几个容器配合就能完全代替 selenium grid（目前仅限chrome和firefox）
*   包含 VNC server（远程桌面），方便远程调试 headless 浏览器
*   全部在 linux 环境下执行，无需设置 windows 节点机，方便自动化
*   方便自定义 Dockerfile ，用户可以自己制作镜像

### 使用 Dockerfile + docker-compose 来创建 docker 镜像并启动

`注意：` 将 Dockerfile 和 docker-compose.yml 放在项目根目录下 

*   `Dockerfile：`

        FROM python:3.6
        RUN pip install scrapy
        RUN pip install selenium
        RUN pip install pymysql

*   `docker-compose.yml：`

        version: "2.0"
        services:
        spider:
            build: ./
            volumes:
            - ./:/code/  # 这里把刚刚的代码映射到这个目录
            working_dir: /code
            command: scrapy crawl chinese_judgment  # 定义启动容器执行的命令
            depends_on:
            - chrome
        chrome:
            # image: selenium/standalone-chrome:latest    
            image: selenium/standalone-chrome:3.8.1 
            ports:
            - "4444:4444"
            shm_size: 2g

*   `启动 docker-compose：`

        docker-compose up -d

*   `查看 docker-compose 生成日志：`

        docker-compose logs

*   `standalone-chrome:3.8.1`

    使用它的原因：

        webdriver.Remote() 不支持 cdp 操作，所以新版的 chrome 不能使用 cdp 来修改，webdriver.navigator 的值。

        这里使用老版的方法可以通过 开发者模型 来对其进行修改



