#  docker 手册

## 注意

1. ubuntu 出现权限不足情况，请使用sudo 管理权限。或者把当前用户加入到docker的用户组中，更新用户组（sudo newgrp docker）

-----

## docker基本操作

```sh
#1 拉取镜像到本地
docker pull 镜像名称[:tag]
```

​	

```sh
#2 查看本地全部镜像
docker images 
```

```sh
#3 删除本地镜像
docker rmi 镜像的标识（部分标识即可，只要是本地所有镜像中可以唯一表示该镜像）
```



```sh
#4 镜像的导入导出
# 将本地的镜像到处
docker save -o 到处的路径 镜像id（镜像标识）
# 加载本地的镜像文件
docker load -i 镜像文件
# 修改镜像名称
docker tag 镜像id（镜像标识） 新镜像名称:版本
```

-----

## 容器的操作

```sh
#1 运行容器
# 简单操作
docker run 镜像标识|镜像名称[:tag]    （若本地没有该镜像，则从默认仓库下载）

# 常用参数
docker run -d -p 宿主机端口:容器端口 --name 容器名称 镜像标识|镜像名称[:tag]
# -d ：代表后台运行容器
# -p 宿主机端口:容器端口  ：为了映射当前Linux的端口和容器的端口
# --name 容器名称 ：指定容器的名称

ex：docker run -d -p 8081:8080 --name tomcat b8
```

```sh
#2 查看正在运行的容器
docker ps [-qa]
# -a：查看全部的容器，包括没有运行
# -q：之查看容器的标识
```

```sh
#3 查看容器的日志
docker logs -f 容器名称（容器id）
# -f：可以滚动查看日志的最后几行
```

```sh
#4 进入容器内部
docker exec -it 容器名称（容器id）bash
```

```sh
#5 删除容器(删除容器前，需要先停止容器)
# 停止指定容器
docker stop 容器id
# 停止全部容器
docker stop $(docker ps -qa)

# 开启容器
docker start 容器id
# 开启全部容器
docker start $(docker ps -qa)

# 删除指定容器
docker rm 镜像id
# 删除全部容器
docker rm $(docker ps -qa)
```

```sh
#6 启动容器
docker start 容器id
```

-----

## Docker 应用

### 准备SSM工程

```
# 注意数据库的用户密码 以及表等信息完整迁移
```



### 准备MySQL容器

```sh
# 运行MySQL容器
docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 daocloud.io/library/mysql:5.7.4

docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root daocloud.io/library/mysql:8.0.2


-----
使用navicat 连接测试(失败，mysql容器无法运行，映射端口出错)
```

### 准备tomcat 容器

```sh
# 运行Tomcat容器，并且把SSM项目的War包部署到Tomcat容器内部
## 可以通过命令将宿主机的内容复制到容器内部
docker cp 文件名称 容器id:容器内部路径
## 举个例子: 
docker cp ssm.war fe:/usr/local/tomcat/webapps
```

###  数据卷

> 为了部署SSM的工程，需要使用CP的命令将宿主机的ssm.war 文件复制到容器内部
>
> 数据卷：将宿主机的一个目录映射到容器的一个目录
>
> 可以在宿主机中操作目录中的内容，那么容器内部映射的文件，也会跟着一起改变

```sh
# 1 创建数据卷
docker volume create 数据卷名称
# 创建数据卷之后，默认回存放在一个目录下 /var/lib/docker/volumes/数据卷名称/_data(在宿主机中创建，并没在容器中生成)

```

---

```sh
# 2 查看数据卷的详细信息
docker volume inspect 数据卷名称
```

----

```sh
# 3 查看全部数据卷
docker volume ls 
```

----

```sh
# 4 删除数据卷
docker volume rm 数据卷名称
```

----

```sh
# 5 应用数据卷
# 当你映射数据卷时，如果数据卷不存在。Docker会帮你自动创建（回将容器内部自带的文件，存储到默认的存放路径）
docker run -v 数据卷名称:容器内部的路径 镜像id
# 直接指定一个路径作为数据卷的存放位置
docker run -v 路径:容器内部的路径 镜像id

#ex:
docker run -d -p 8081:8080 --name tomcat_demo1 -v /home/andy/myself/docker_files/tomcat/demo1:/usr/local/tomcat/webapps b8


# 需要在数据卷目录中加入 名称为：ROOT 的文件夹，并在文件夹中放入index.HTML 的文件才可以访问（使用127.0.0.1:8081来访问）
```

## Docker自定义镜像

> 中央仓库上的镜像，也是Docker的用户自己上传上去的

```sh
#1 创建一个Dockerfile文件，并制定自定义镜像（Dockerfile文件是一个最基本的文本文件，但是其没有任何包括.txt 等后缀名，是没有后缀的）
# Docklerfile 文件中常用的内容
from:	指定当前自定义镜像以来的环境
copy: 	将相对路径（当前Dockerfile的路径为根）下的内容复制到自定义镜像中
workdir: 	声明镜像的默认工作目录
cmd: 	需要执行的命令(在workdir 下执行的，cmd可以写多的，只以最后一个为准)

# 举个例子，自定义一个tomcat_demo1 镜像，并且将indexhtml部署到tomcat中
from daocloud.io/library/tomcat:8.5.15-jre8
copy index.html  /usr/local/tomcat/webapps/ROOT

```

---

```sh
#2 将准备好的Dockerfile和相应的文件copy到Linux系统中，通过Docker的命令制作Docker镜像（此处的镜像名称为自定义镜像名称，tag版本号也是自定义版本号）
docker build -t 镜像名称:[tag] .

# ps: 不要忘记最后的点（.）这是代表当前目录
```

## Docker-Compose

> 之前运行一个镜像，需要添加大量的参数
>
> 可以通过Docker-Compose编写这些参数
>
> Docker-Compose可以帮助我们批量的管理这些容器
>
> 只需要通过一个docker-compose.yml 文件去维护即可。

### 下载Docker-Compose

```sh
#1  去github下载docker-compose

#2 复制docker-compose到linux系统

#3 对文件重命名，并修改其权限，使其成为可执行文件
1. 	mv 文件名 新文件名
2. 	chmod 777 docker-compose

# 4 添加为环境变量
# 4.1 移动文件到 bin目录下
mv docker-compose bin
# 4.2 先cd bin 移动到当前用户的bin目录， 并使用pwd 获取当前绝对路经 

# 4.3 修改 /etc/profile 中最底下 加入 刚刚pwd获取到的绝对路径（该PATH以冒号为每个环境变量的分隔符）：
export PATH=xxx:xxxx:XXX
# 4.4 重新加载profile文件 
source /etc/profile 
```

![docker-compose运行图](https://gitee.com/andyxiaopeng/picbed/raw/master/pic/20201027162307.png)

### docekr-compose来管理mysql和tomcat容器

> 设定yml文件
>
> ​	yml文件以key: value方式来制定配置信息(不要忘记冒号后面还有**空格**)
>
> ​	多个配置信息以换行+缩进的方式来区分

```yml
#举个例子(yml的缩进是两个空格 而不是两个制表符tab)
version: '3.1'
services:  
  mysql:                        # 服务的名称
    restart: always      # 代表只要docker 启动，那么这个容器就跟着一起启动
    image: daocloud.io/library/mysql:8.0.1     # 指定镜像
    container_name: mysql # 指定容器名称
    ports: 
      - 3306:3306     # 指定端口号的映射
    environment:  
      MYSQL_ROOT_PASSWORD: root     # 指定mysql 的root用户 的登录密码
      TZ: Asia/Shanghai                                 # 指定时区
    volumes: 
      - /opt/docker_mysql_tomcat/mysql_data:/var/lib/mysql         # 映射数据卷
  tomcat: 
    restart: always
    image: daocloud.io/library/tomcat:8.5.15-jre8
    container_name: tomcat
    ports: 
      - 8081:8080
      - 9988:9988
    environment: 
      TZ: Asia/Shanghai
    volumes: 
      - /opt/docker_mysql_tomcat/tomcat_webapps:/usr/lib/webapps
      - /opt/docker_mysql_tomcat/tomcat_logs:/usr/local/tomcat/logs
      
```

### 使用docker-compose命令管理容器

> 在使用docker-compopse 的命令时候，默认会在当前目录下找docker-compose.yml文件

```sh
#1 基于docker-compose.yml 启动管理的容器
docker-compose up -d

```

![docker-compose成功启动mysql和tomcat服务](https://gitee.com/andyxiaopeng/picbed/raw/master/pic/20201027162315.png)

----

```sh
#2 关闭并删除容器
docker-compose down
```

----

```sh
#3 开启|关闭|重启 已经存在的由docker-Compose维护的容器
docker-compose start|stop|restart
```

-------

```sh
#4 查看docker-compose管理的容器
docker-compose ps
```

----

```sh
#5 查看日志
docker-compose logs -f
```

###  docker-compose 配合dockerfile使用

> 使用docker-compose.yml 文件以及Dockerfile文件生成自定义镜像的同时启动当前镜像，并且由docker-compose去管理

[docker-compose.yml]()

```yml
# yml 文件
version: '3.1'
services: 
  ssm: 
  restart: always
  build:                # 构建自定义镜像
    context: ../    # 指定dockerfile文件的所在路径
    dockerfile: Dockerfile   # 指定Dockerfile文件名称
  image: ssm:1.0.1
  container_name: ssm
  ports: 
    - 8082:8080
  environment: 
    TZ: Asia/Sahnghai
```

[Dockerfile]()

```sh
from daocloud.io/library/tomcat:8.5.15-jre8
copy index.html  /usr/local/tomcat/webapps/ROOT
```

![使用docker-compose启动自制镜像](https://gitee.com/andyxiaopeng/picbed/raw/master/pic/20201027162323.png)

![警告分析](/home/andy/.config/Typora/typora-user-images/image-20200826010248330.png)

```sh
# 此处警告说明镜像以前未存在，因此这次以及build出来，以后再次使用该镜像，则不会重新通过Dockerfile构建，除非你自己手动输入 “dockercompose build”或者“docker-compose up --build”来重新构建镜像
```

[访问效果]()

![image-20200826010859351](https://gitee.com/andyxiaopeng/picbed/raw/master/pic/20201027162328.png)



## Docker Cl、CD

> 项目部署
>
> ​	1、将项目通过maven进行编译打包
>
> ​	2、将文件上传到制定的服务器
>
> ​	3、将war包放到tomcat的目录中
>
> ​	4、通过dockerfile将toncat和war包转成一个镜像，由dockercompose去运行容器
>
> 项目更新
>
> ​	将上述功能重复

### CI介绍

> ci（continuous intergration）持续集成
>
> 持续集成：编写代码时，完成一个功能后，立即提交代码到git仓库，将项目重新的构建并且测试。
>
> - 快速发现错误
> - 防止代码偏离主分支

### 实现持续集成

#### 搭建gitlab服务器

> 1. 创建一个全新的虚拟机，并且指定至少4G的运行内存
> 2. 安装docker以及dockercompose
> 3. **docekrcompose.yml**文件去安装gitlab（首次下载需要下载一个多G镜像，谨慎！！！）
> 4.  注意需要避免SSH的22端口冲突（修改/etc/ssh/sshd_config 文件，然后重启sshd服务   systemctl restart sshd）

```yml
versoin: '3.1'
services: 
  gitlab: 
    image: 'twang2218/gitlab-ce-zh:11.1.4'
    container_name: "gitlab"
    restart: always
    privileged: true
    hostname: 'gitlab'
    environment: 
      TZ: 'Asia/Shanghai'
      GITLAB_OMNIBUS_CONFIG:    
        external_url 'http://192.168.190.110'
        gitlab_rails['time_zone'] = 'Asia/Shanghai'
        gitlab_rails['smtp_enable'] = true
        gitlab_rails['gitlab_shell_ssh_port'] = 22
    ports: 
      - '80:80'
      - '443:443'
      - '22:22'
    volumes: 
      - /opt/docker_gitlab/config:/etc/gitlab
      - /opt/docker_gitlab/data:/var/opt/gitlab
      - /opt/docker_gitlab/logs:/var/log/gitlab
```

#### 搭建GitLab-Runner

> 搭建过程非常复杂，尚未整理清楚

#### 整合项目入门测试

> 1. 创建maven工程，添加一个web.xml文件，编写html页面
>
> 2. 编写gitlab-ci.yml文件
>
>    ```yml
>    stages: 
>      - test
>    test: 
>      stage: test
>      script: 
>        - echo first test ci                  #输入所需要的命令
>    ```
>
>    
>
> 3. 将maven工程推送到gitlab中
>
> 4. 可以在gitlab中查看到gitlab-ci.yml编写的内容

#### 编写.gitlab-ci.yml 文件

> 1. 编写.gitlab-ci.yml测试命令
>
>    ```
>    stages: 
>      - test
>    test: 
>      stage: test
>      script: 
>        - echo first test ci                  #输入所需要的命令
>        - /usr/local/maven/apache-maven-3.6.3/bin/mvn package
>    
>    ```
>
>    
>
> 2. 编写关于dockerfile 以及cocker-compose.yml文件的具体内容
>
>    ```yml
>    # Dockerfile
>    FROM daocloud.io/library/tomcat:8.5.15-jre8
>    COPY testci.way /usr/local/tomcat/webapps
>    ```
>
>    -----------
>
>    **docker-compose.yml**
>
>    ```yml
>    version: '3.1'
>    services: 
>      testci: 
>        build: docker
>        restart: always
>        container_name: testci
>        ports: 
>          - 8080:8080
>    ```
>
>    ---------
>
>    **.gitlab-ci.yml**
>
>    ```yml
>    stages: 
>      - test
>    test: 
>      stage: test
>      script: 
>        - echo first test ci                  #输入所需要的命令
>        - /usr/local/maven/apache-maven-3.6.3/bin/mvn package
>        - cp target/testci-1.0-SNAPSHOT.war docekr/testci.war
>        - docekr-compose down
>        - docker-compose up -d --build
>        -dopcker rmi $(docker images -qf dangling=true)
>        
>    ```
>
> 3. 测试

### CD介绍

> CD ： 持续交付 持续部署
>
> 持续交付：将代码交付给专业的测试团队去测试
>
> 持续部署：将测试通过的代码，发布到生产环境

### 实现持续交付　持续部署

#### 安装Jenkins

> 官网：https://www.jenkins.io/

```yml
version: '3.1'
services:
  jenkins:
    image: jenkins/jenkins
    restart: always
    container_name: jenkins
    ports: 
      - 8888:8080
      - 50000:50000
    volumes:
      - ./data:/var/jenkins_home
```

> 第一次运行时，会因为data目录没有权限而启动失败
>
> ```sh
> chmod 777 data
> ```
>
> 访问：ip：8888

> 输入密码
>
> 需要查看日志才能获取密码
>
> docker-compose logs -f

> 手动指定插件安装
>
> publish ssh....
>
> git param....

#### 配置目标服务器以及GItlab免密码登录

> GitLab -> Jenkins -> 目标服务器

> 

