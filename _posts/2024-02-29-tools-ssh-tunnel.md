---
title: docker的使用方法
layout: command
categories: 工具
tags: docker
excerpt: 关于docker的相关用法详解
---

Docker 是一种容器化平台，可以让开发人员将应用程序及其所有依赖性打包成一个独立的容器，以便在任何环境中快速部署和运行。它通过利用容器技术，实现了应用程序与底层系统的解耦，使得应用程序可以在任何支持 Docker 的环境中轻松部署。


## wsl2安装docker

接下来添加Docker源：
依次执行如下命令：
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt update
```
配置完成软件源之后下一步是安装Docker，命令如下：
```
sudo apt install -y docker-ce
```



开启或关闭防火墙

要打开防火墙，请在终端输入 `sudo ufw enable`。要关闭 **ufw**，请输入 `sudo ufw disable`。
允许或阻止特定网络活动

很多程序专用于提供网络服务。例如，您可共享内容或允许其他人远程查看您的桌面。根据您安装的附加程序，您可能需要调整防火墙设置，以允许这些服务正常运行。UfW 附带了很多已预配置好的规则。例如，要允许 SSH 连接，请在终端输入 sudo ufw allow ssh。要阻止 ssh，请输入 sudo ufw block ssh。

提供服务的每个程序都使用特定的网络端口。要允许访问该程序的服务，您可能需要允许访问在防火墙上为其分配的端口。要允许连接端口 53，请在终端输入`sudo ufw allow 53`。要阻止端口 53，请输入 `sudo ufw block 53`。

要检查 ufw 的当前状态，请在终端输入 `sudo ufw status`。






## 启动docker2种方法
```
1.sudo service docker start   

  sudo service docker stop
  
  sudo service docker status


2.systemctl start docker

  systemctl stop docker

  systemctl restart docker

  systemctl status docker

  systemctl enable docker

```



**查看docker信息**

```
docker version
```

**查看docker镜像**

```
docker images
```

**查看docker运行的容器(-a是指运行的所有容器)**

```
docker ps -a
```

**拉取nginx镜像  镜像后面不指定版本默认为latest**

```
docker pull nginx 
```

**保存nginx镜像  [镜像名]:[版本]**

```
docker save -o nginx.tar nginx:latest
```

**删除nginx镜像  [镜像名]:[版本]**

```
docker rmi nginx:latest
```

**恢复nginx镜像  [镜像名]:[版本]**

```
docker load -i nginx:latest
```

**如果没有拉取MySQL镜像 会执行拉取MySQL镜像然后创建一个mysql2(容器名)的容器**

```
docker run -d \                      	#创建并运行一个容器，-d是让容器在后台运行
  --name mysql \					 	#给容器起个名字，必须唯一
  -p 3306:3306 \						#设置端口映射   [宿主机]:[容器内部]   宿主机端口映射到容器内端口
  -e TZ=Asia/Shanghai \					#KEY=VALUE   设置环境变量 
  -e MYSQL_ROOT_PASSWORD=123 \			
  mysql:latest							#指定运行镜像名字   [镜像名]:[版本]  默认latest可以省略，指定版本必须写

docker run -d --name mysql -p 3306:3306 -e TZ=Asia/Shanghai -e MYSQL_ROOT_PASSWORD=123456 mysql:latest							

docker run --name nginx -d -p 80:80 nginx:latest
```

**停止容器**  

```
docker stop mysql2
```

**启动容器**

```
docker start mysql
```

**删除容器**

```
docker rm -f mysql
```

**查看nginx容器运行日志**

```
docker logs -f nginx
```

**进入nginx容器内部**

```
docker exec -it nginx bash
```

**进入MySQL并且连接**

```
docker exec -it mysql mysql -uroot -p 
```

**查看nginx容器内的ip**

```
docker inspect nginx | grep IPAddress
```

**配置命令的别名**

```
vim ~/.bashrc
```

**让 ./bashrc 生效**

```
source ~./bashrc
```





## 今天在这里讲如何在docker上运行nignx镜像，并将配置文件和目录挂载到宿主机上，以实现方便统一的管理配置信息。
```
mkdir -p /usr/local/nginx/conf
mkdir -p /usr/local/nginx/logs
mkdir -p /usr/local/nginx/html
```
## 下面需要先运行容器，方便把文件本来的内容拷贝出来，然后再将容器删除，因为自己手动创建的配置文件容易有语法错误，当然如果你有了争取的配置文件也可以直接使用，就不需要创建容器拷贝出来后再删除这个操作了。接下来几个步骤可以跳过
	
**1. 先用 nginx 镜像创建 nginx 容器，将需要挂载的文件拷贝出来**

```
docker run --name nginx -d -p 80:80 nginx
```
 
**2. 将容器中的 nginx.conf 文件拷贝到宿主机中**

```
docker cp nginx:/etc/nginx/nginx.conf /usr/local/nginx/conf/nginx.conf
```
 
**3. 将容器中 conf.d 文件夹（包括里面的文件）拷贝到宿主机中**

```
docker cp nginx:/etc/nginx/conf.d /usr/local/nginx/conf/conf.d
```
 
**4. 将容器中的 html 文件夹拷贝到宿主机中**

```
docker cp nginx:/usr/share/nginx/html /usr/local/nginx/
```

**5.删除正在运行的容器容器(-f 的参数作用是强制删除)**

```
docker rm -f nginx
```

**最终可以在宿主机中看到这些目录和文件夹，并且其中的html中包含了html文件，conf文件夹中包含了配置文件。**


**全部准备好后，做最终的文件夹挂载，端口映射**

## 运行启动命令，并将端口进行映射，文件进行挂载。

```
docker run -p 80:80 --name nginx -v /usr/local/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /usr/local/nginx/conf/conf.d:/etc/nginx/conf.d -v /usr/local/nginx/logs:/var/log/nginx -v /usr/local/nginx/html:/usr/share/nginx/html -d --restart=always nginx:latest
```

#格式化后的代码

```
docker run -p 80:80 --name nginx \
-v /usr/local/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /usr/local/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /usr/local/nginx/logs:/var/log/nginx \
-v /usr/local/nginx/html:/usr/share/nginx/html \
-d \
--restart=always \
nginx:latest

 
1.--name是设置容器名
2.-p是容器与宿主机的端口映射
3.-v是做卷挂载，实质上就是文件的映射
4.-d是后台运行
5.--restart 是Docker提供重启策略控制容器退出时或Docker重启时是否自动启动该容器。，always表示docker重启后，这个容器会自动重启
```

**执行完成后，在浏览器查看是否可以访问。**


## 以上就是docker运行nginx的所有步骤了，如果要配置ssl的话，需要先去域名申请证书，再配置到配置文件中，docker的操作步骤不影响。不过以上要注意几个问题

    容器的端口要映射出来才可以访问，如果是在阿里云服务器上，还需要把阿里云的对应的端口开通
    如果部署的是前端系统，需要把前端文件放到挂载的文件夹中，且nginx配置的访问路径是容器中对应的路径，不要配置成宿主机中的路径，否则会访问不到的
