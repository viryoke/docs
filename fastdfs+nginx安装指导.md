# 海报服务器安装步骤(单服务器)

## 一、搭建FASTDFS

### 1、安装libfastcommon

unzip libfastcommon-master.zip   

cd libfastcommon-master

./make.sh

./make.sh install

### 2、安装fastdfs

unzip fastdfs-master.zip

cd fastdfs-master

./make.sh

./make.sh install

**服务脚本地址如下**

/etc/init.d/fdfs_storaged

/etc/init.d/fdfs_trackerd

**配置文件地址如下**

/etc/fdfs/client.conf.sample

/etc/fdfs/storage.conf.sample

/etc/fdfs/tracker.conf.sample

**命令行工具地址如下**

/usr/bin/fdfs_appender_test

/usr/bin/fdfs_appender_test1

/usr/bin/fdfs_append_file

/usr/bin/fdfs_crc32

/usr/bin/fdfs_delete_file

/usr/bin/fdfs_download_file

/usr/bin/fdfs_file_info

/usr/bin/fdfs_monitor

/usr/bin/fdfs_storaged

/usr/bin/fdfs_test

/usr/bin/fdfs_test1

/usr/bin/fdfs_trackerd

/usr/bin/fdfs_upload_appender

/usr/bin/fdfs_upload_file

### 3、配置tracker服务器

**复制样例配置文件并重命名**

cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf

**创建数据文件存放目录**

mkdir -p /opt/fdfs/tracker

**编辑配置文件**

vim /etc/fdfs/tracker.conf

**修改的内容如下**

disabled=false             # 启用配置文件

port=22122                 # tracker服务器端口（默认22122）

base_path=/opt/fdfs/tracker # 存储日志和数据的根目录，自行规划

**启动tracker服务器**

/etc/init.d/fdfs_trackerd start

**初次启动，会在/opt/fdfs/tracker目录下生成logs、data两个目录**

/opt/fdfs/tracker/data

/opt/fdfs/tracker/logs

### 4、配置storage服务器

**复制样例配置文件并重命名**

cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf

**创建数据文件存放目录**

mkdir -p /opt/fdfs/storage

**编辑配置文件**

vim /etc/fdfs/storage.conf

**修改的内容如下**

disabled=false                      # 启用配置文件

port=23000                          # storage服务端口

base_path=/opt/fdfs/storage          # 数据和日志文件存储根目录

store_path0=/opt/fdfs/storage        # 第一个存储目录

tracker_server=ip01:22122 #tracker服务器IP和端口，根据实际填写

http.server_port=8888               # http访问文件的端口

**启动storage服务器**

/etc/init.d/fdfs_storaged start

**初次启动，会在/opt/fdfs/storage目录下生成logs、data两个目录**

/opt/fdfs/storage/data

/opt/fdfs/storage/logs

### 5、文件上传测试

**修改tracker服务器客户端配置文件**

cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf

vim /etc/fdfs/client.conf

**修改以下配置，其它保持默认**

base_path=/opt/fdfs/tracker

tracker_server=ip01:22122 #请填写实际的tracker服务器ip地址

**执行文件上传命令**

/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /opt/test.png

返回文件ID号：group1/M00/00/00/tlxkwlhttsGAU2ZXAAC07quU0oE095.png

（能返回以上文件ID，说明文件已经上传成功）

## 二、搭建NGINX

### 1、解压fastdfs-nginx-module-master.zip

cd /opt

unzip fastdfs-nginx-module-master.zip    

### 2、安装nginx(添加fastdfs-nginx-module模块)

tar –zxvf nginx-1.13.1.tar.gz

cd nginx-1.13.1

./configure --add-module=/opt/fastdfs-nginx-module/src

make & make install

### 3、进行相关文件的配置

**复制fastdfs-nginx-module源码中的配置文件到/etc/fdfs目录并修改**

cp /opt/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/

vi /etc/fdfs/mod_fastdfs.conf

**修改以下配置:**

connect_timeout=10

base_path=/tmp

tracker_server=ip01:22122

storage_server_port=23000

group_name=group1

url_have_group_name=true

store_path0=/opt/fdfs/storage

**复制fastdfs-master的部分配置文件到/etc/fdfs目录**

cd /opt/fastdfs-master/conf

cp http.conf mime.types /etc/fdfs/

**在/opt/fdfs/storage文件存储目录下创建软连接,将其链接到实际存放数据的目录**

ln -s /opt/fdfs/storage/data/ /opt/fdfs/storage/data/M00

**配置 nginx**

user nobody;

worker_processes 1;

events {

​    worker_connections1024;

}

http {

​    include mime.types;

​    default_typeapplication/octet-stream;

​    sendfile on;

​    keepalive_timeout 65;

​    server {

​        listen 8888;

​        server_namelocalhost;

​        location~/group([0-9])/M00 {

​           ngx_fastdfs_module;

​        }

​        error_page 500 502503 504 /50x.html;

​        location =/50x.html {

​            root html;

​        }

​    }

}

**8888端口值是要与/etc/fdfs/storage.conf中的http.server_port=8888 相对应, 因为http.server_port默认为 8888,如果想改成 80,则要对应修改过来。**

**storage对应有多个group的情况下,访问路径带group名,如/group1/M00/00/00/xxx,对应的nginx配置为:**

location ~/group([0-9])/M00 {

​        ngx_fastdfs_module;

}

**如查下载时如发现老报404,将nginx.conf第一行usernobody修改为user root后重新启动;**

### 4、启动nginx

/usr/local/nginx/sbin/nginx

(重启 nginx 的命令为:/usr/local/nginx/sbin/nginx -sreload)

### 5、测试nginx

通过浏览器访问测试时上传的文件 http://ip:port/group1/M00/00/00/tlxkwlhttsGAU2ZXAAC07quU0oE095.png
