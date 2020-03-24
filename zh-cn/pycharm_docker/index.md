# Pycharm远程调试Docker


使用学校集群跑深度学习代码，终端连接调试过于繁琐，所以有了本文。

## 准备
- Ubuntu 16.04(远程服务器)
- Mac或Ubuntu(本地)
- docker(远程服务器)
- openssh-server(远程服务器)
- Pycharm profession版(本地)
- ssh(本地)
## 原理
本地利用SSH链接远程服务器交互数据，Pycharm调用远程docker容器中的python编译器运行代码，并在本地显示远程结果。
## 配置流程
1. 在远程服务器创建docker container
1. 远程服务器ssh服务配置
1. Pycharm链接远程服务器(文件同步)
1. Pycharm链接远程的docker container (配置远程编译器)
 
## 一、远程服务器创建docker container
在创建容器时需要设置端口映射，-p 本机端口：容器端口
~~~shell
docker run -id -p 8022:22 --name container_name -v 宿主机目录:容器目录 image_name:tag
~~~
~~~markdown
参数解释：
    -id：使容器退出时不会关闭，如果为-it 则退出容器后，容器会停止
    -p ：端口映射，将容器的22端口映射到服务器的8022端口，22端口为sftp的端口，远程时必须开放22端口才能保证文件的上传下载同步
    -v ：目录的挂载，将服务器与容器的目录同步
~~~
## 二、 远程服务器ssh服务配置
接下来需要配置远程服务器的ssh服务。
### 第一步，我们需要在远程服务器上安装openssh-server
~~~shell
$ apt update && apt install openssh-server
~~~
{{% admonition "tip" "注意" %}}
这里注意，不仅仅是远程服务器上要安装ssh服务，同时远程服务器上的docker container也内也需要安装openssh-server。
{{% /admonition %}}

### 第二步，安装完成以后需要配置ssh服务
~~~shell
# 次配置在docker container中完成
$ echo 'root:test' | chpasswd
# 将Root的密码修改为test
$ sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# 允许使用root身份登录
$ sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
$ echo "export VISIBLE=now" >> /etc/profile12345678
~~~
{{% admonition "warning" "注意：" %}}
其实直接在容器中输入 passwd 然后输入密码即可，通常docker容器下的用户为root，其他sshd_config配置实测没有必要，可能是新版本openssh默认根用户登录
{{% /admonition %}}

### 第三步，配置好ssh服务之后重启ssh服务
~~~shell
$ service ssh restart
~~~
### 第四步，测试docker container中ssh服务端口在远程服务器上的映射
~~~shell
# 此操作在远程服务器
$ docker port <your container name> 22
# 此操作将查看docker container中端口22，在远程服务器上端口的映射
# 输出结果如下所示
0.0.0.0:8022
# 表明只要ssh链接远程服务器的8022端口，实际是链接docker container中的22端口。
~~~
### 第五步，测试是否能够使用ssh链接docker container
~~~shell
$ ssh root@<你服务器的ip地址> -p 8022
# 密码就是刚刚重新设置的
~~~
如果能够链接成功到docker container就完成了此次ssh的配置。
{{% admonition "tip" "注意" %}}
如果失败请按以下顺序检查
{{% /admonition %}}

1. ssh的端口配置是否正确？（包括服务器和docker container）
2. 是否开启了防火墙，将端口禁用？ 
3. 也可以运行 sudo gedit /etc/ssh/sshd_config 修改配置，保存，并重起ssh 
~~~shell
sudo service ssh restart
~~~
到这里远程的配置到一段落，接下来使用本地的pycharm操作
Pycharm链接远程docker container(文件同步)

***

## 配置Pycharm
### 配置SFTP
在导航栏中 Tools>Depolyment>Configuration中添加配置SFTP。
如图
![Local Picture](/images/Apple-Devices-Preview.png)
添加配置SFTP，点击弹窗左上角的+号。选择SFTP，根据自己的实际情况进行配置。
![Local Picture](/images/Apple-Devices-Preview.png)
PS：这里的root密码就是之前设置好的test
### 配置SFTP中的mapping
都配置完之后。打开自动上传功能
Tools>Depolyment>Automatic Upload(always)
本地修改好代码只要按保存键就自动将本地代码上传至远程docker container中。
到这里已经配置好代码的自动同步了。还差最后一步，远程调试就配置成功。
### Pycharm链接远程docker container (配置远程编译器)
打开Pycharm专业版的配置
添加新编译器(远程docker container编译器)
在打开的页面选择之前配置好的SFTP

{{% admonition "tip" "注意" %}}
通常选择完之后羡慕有两个选项
Create: 新建SFTP
Move: 将选择的SFTP作为编译器的SFTP
通常选择Move就好
{{% /admonition %}}


最后配置docker container的编译器位置，还有项目位置的映射。
完成这一步就彻底搞定Pycharm远程调试Docker container
只要在调试的时候，选择新建的远程调试编译器就好



本文参考至：
[PyCharm+Docker：打造最舒适的深度学习炼丹炉](https://zhuanlan.zhihu.com/p/52827335)
[pycharm远程调试docker containers](https://www.cnblogs.com/ruiyang-/p/10158658.html)

