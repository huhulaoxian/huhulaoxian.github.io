# Docker常用命令及注意事项

## Docker的常用命令
### 服务相关：
* 启动
~~~shell
systemctl start docker
~~~
* 停止
~~~
systemctl stop docker
~~~
* 查看状态
~~~
systemctl status docker
~~~
* 重启
~~~
systemctl restart docker
~~~
* 设置开机启动
~~~
systemctl enable docker
~~~
### 镜像相关：
* 查看镜像
~~~
docker images
~~~
列出所有镜像的id
~~~
docker images -q
~~~
* 搜索镜像
~~~
docker search image_ name
~~~
* 拉取镜像
~~~
docker pull image_name 
# 指定版本号
docker pull image_name:version 
~~~
* 删除镜像
~~~
docker rmi image_name or image_id
~~~
{{% admonition "warning" "注意：" %}}
最好使用 `image_name：tag` 的形式,否则会存在子镜像依赖的情况，导致删除不了。
{{% /admonition %}}

### 容器相关：
* 查看正在运行的容器
~~~
docker ps
# 查看所有容器
docker ps -a
~~~
* 创建
~~~
docker run -it or -id --name container_name image_name:version /bin/bash
~~~
~~~markdown
-it:交互式容器，退出容器后容器自动关闭。t：分配一个伪终端
-id：守护式容器，退出容器后容器不会关闭
-p：开放端口 通常 8022:22 意为将容器的22号端口与服务器8022端口绑定
/ bin/bash: 直接进入容器
~~~
* 进入
~~~
# 退出终端后不会停止容器：
docker exec container_name or id
# 退出容器后容器会停止：
docker attach container_name or id
~~~
* 启动
~~~
docker start  container_name or id
~~~
* 停止
~~~
docker stop container_name or id
~~~
* 删除
~~~
docker rm container_name or id
~~~
* 查看容器信息 
~~~
docker inspect container_name or id 
~~~
### 数据卷相关：
* 创建容器时，使用-v参数设置数据卷
~~~
docker run -it --name container_name -v 宿主机目录：容器目录 image_name
~~~
* 注意事项
	a. 目录必须是绝对路径
	b. 如果目录不存在会自动创建
	c. 可以挂载多个数据卷
	d. 一个数据卷也可以被多个容器挂载
### 镜像制作：
* 容器转镜像，数据卷挂载的数据不会被打包，其余文件都会
~~~
docker commit 容器id  镜像名称:版本号
docker save -o 压缩文件名称 镜像名称:版本号
docker save 镜像名称:版本号 -o /home/myubuntu-save-1204.tar
~~~
* 加载保存的镜像:
~~~
docker load -i压缩文件名称 
docker load -i /home/myubuntu-save-1204.tar
~~~
* dockerfile制作镜像
~~~
docker build -f dockerfile_path -t image_name 
~~~
~~~markdown
-f:指定dockerfile路径
-t:指定新的镜像名字
 .:上下文路径。
~~~
   * 由于 docker 的运行模式是 C/S。我们本机是 C，docker 引擎是 S。实际的构建过程是在 docker 引擎下完成的，所以这个时候无法用到我们本机的文件。这就需要把我们本机的指定目录下的文件一起打包提供给 docker 引擎使用。如果未说明最后一个参数，那么默认上下文路径就是 Dockerfile 所在的位置。 
   
 
## 常见问题：
### 1. 在容器中执行apt update的时候可能会出现0% working 的问题
”这里不是源的问题，因为容器环境太过纯净，这里需要安装apt-transport-https这个deb文件，下载的时候也要注意不要下载最新的版本，否则也会出现依赖问题，要下载和当前docker容器内的apt相匹配的版本。“


