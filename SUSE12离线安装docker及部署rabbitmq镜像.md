# SUSE12离线安装docker及部署rabbitmq镜像

## 一、离线安装docker

### 1、下载docker-17.06.2-ce.tgz

### 2、解压docker-17.06.2-ce.tgz

```
tar zxvf docker-17.06.2-ce.tgz
```

### 3、复制文件到/usr/bin/

```
cp docker/* /usr/bin/ 
```

### 4、启动docker daemon

```
dockerd &
```

## 二、离线导入rabbitmq镜像并部署

### 1、事先导出rabbitmq镜像（已导出，可忽略）

```
docker pull rabbitmq:alpine
docker save rabbitmq:alpine > /opt/rabbitmq.tar
```

### 2、导入rabbitmq镜像

```
docker load < /opt/rabbitmq.tar
```

### 3、查看是否导入成功

```
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
rabbitmq            alpine              04cf057cce54        26 hours ago        38.7 MB
```

### 4、部署rabbitmq

```
docker run --name rabbitmq -p 5672:5672 -p 15672:15672 -d rabbitmq:alpine
```

### 5、查看是否部署成功

```
docker ps
```

### 6、配置rabbitmq

```shell
# 执行命令进入rabbitmq容器内
docker exec -it rabbitmq bash
# 执行rabbitmqctl命令进行相关配置
rabbitmqctl add_user admin admin
......
# 若想web访问rabbitmq管理界面，需开启rabbitmq_management插件,然后重启rabbitmq容器
rabbitmq-plugins enable --offline rabbitmq_management
exit
docker restart rabbitmq
```
