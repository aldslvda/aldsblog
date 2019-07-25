title: 运行多容器的应用--使用docker-compose
date: 2019-06-03 16:57:00
tags:
- docker
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/docker.png?raw=true"
categories:
- 随手摘录
toc: true
comment: true
---


> 使用docker-compose.yml对多容器的应用(组)进行容器编排

<!-- more -->

使用docker-compose.yml来定义你的应用，并且使用up/down命令来运行，这样可以让你对多容器的应用进行编排。     
Compose是一个定义和运行多容器应用的工具, 也就是容器编排工具。使用Compose时，你会使用到一个Compose配置文件去配置你的应用的各个服务。然后通过使用这个配置文件就可以一条命令启动所有的服务。

# 和docker命令的类似之处
Docker-compose命令大体上和docker命令类似，除了一些给多容器应用的使用的附加指令。尤其是下面这些命令，和docker命令很类似:

## docker-compose up / down
docker-compose up 用于创建并运行容器. 在detached( -d )模式下, 启动容器之后Compose会退出, 但是容器还在后台运行。

```bash
docker-compose up -d rabbit-mq
```
docker-compose down 用于停止并移除容器，网络，镜像和卷。
  	
```bash
docker-compose down -v
```
这个命令有一些有用的选项:   

- --rmi [type] 移除镜像。  
  type必须是以下其中一个:
    - 'all': 移除任何服务使用的所有的镜像。
    - 'local': 只移除没有通过`image`设置自定义标签的镜像。
- -v, --volumes 移除在配置文件中通过`volumes`声明的命名卷和attach到这个容器的所有卷。
- --remove-orphans   移除没有在Composer中定义的容器(杀死孤儿)。

# 创建一个Dockerfile
就像运行一个docker需要镜像一样，创建镜像也需要一个dockerfile来确定怎样创建镜像和容器。一个Ruby on Rails应用使用的Dockerfile可能是这样:
```dockerfile
FROM ruby:2.3.3
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN mkdir /myapp
WORKDIR /myapp
COPY Gemfile /myapp/GemfileCOPY Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
COPY . /myapp
```

[docker-compose: getting started guide](https://docs.docker.com/compose/gettingstarted/) 有一步一步的指导教你创建dockerfile然后把dockerfile改写成docker-compose.yml。

# docker-compose.yml之旅: 结构和组成
要定义你自己的多容器应用，你需要在你的应用的root文件夹使用docker-compose.yml。    
compose-file的文档提供了详尽的解释和指导去生成这个文件。 下面是主要特性的快速上手指南:    
文件的最开始引入版本: ```version: "3"```     
然后定义服务: ```services:```     
之后列出你想要创建的服务,  每一个容器都有对应的配置项。     

- image - 指定一个dockerhub上或者本地有的镜像   
```image: ruby:alpine```
- depends_on — 这个容器依赖的其他容器，启动父容器也会同时启动父容器依赖的容器。
```
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
```
- environment — 增加环境变量.
```
environment:  
  RACK_ENV: development  
  SHOW: 'true'  
  SESSION_SECRET:
environment:  
  - RACK_ENV=development   
  - SHOW=true  
  - SESSION_SECRET
```
- volumes — 包含宿主机的路径或者命名卷。 你可以通过这样的格式替换容器的数据 ```<path to file>:<path to location in container>``` 并且可以把文件夹权限接在后面``` :ro``` (read only)
```dockerfile
version: "3.2"
services:
  web:
    image: nginx:alpine
    volumes:      
	  - type: volume        
	    source: mydata        
		target: /data        
		volume:          
		nocopy: true      
	  - type: bind        
	    source: ./static        
		target: /opt/app/static
  db:    
    image: postgres:latest    
	volumes:      
	  - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"      
	  - "dbdata:/var/lib/postgresql/data"
```
- ports — 可以指定映射两个对应的端口(HOST:CONTAINER), 或者只指定容器的端口，系统会自动分配一个临时的外部端口。   
```dockerfile
ports: 
  - "3000" 
  - "3000-3005" 
  - "8000:8000" 
  - "9090-9091:8080-8081" 
  - "49100:22" 
  - "127.0.0.1:8001:8001" 
  - "127.0.0.1:5000-5010:5000-5010" 
  - "6060:6060/udp" 
```

Docker Compose 功能非常强大，还有更多用法需要学习，Docker文档有非常详尽的说明，本文只是略作抛砖引玉。
