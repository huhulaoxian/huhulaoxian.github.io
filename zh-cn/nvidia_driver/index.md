# Ubuntu下Nvidia显卡驱动安装

本文介绍了ubuntu通过英伟达官网安装显卡驱动的方法，当然也可以使用ubuntu图形界面中的“软件和更新”进行安装，但是不推荐
<!--more-->

本来不想记这个的，ubuntu图形界面中的‘软件和更新’就可以办到，它会自动检索推荐版本，但不是最新的驱动版本。
由于要学习Pytorch 1.4（目前的最新版本），需要安装cuda 9.2 ，所以显卡驱动版本必须大于396.26（使用nvidia-smi命令查询显卡驱动版本）。ubuntu推荐的显卡驱动就不行了。只能老老实实安装通过英伟达官网下载安装最新驱动了，这也是最为保险和妥当的方式。
## 环境配置
* 系统：ubuntu 16.04
* 显卡：GTX 850M
## 安装步骤
### 第一步、英伟达官网查询自己显卡型号的驱动并下载.run文件
### 第二步、卸载旧的英伟达驱动
~~~shell
sudo apt-get remove --purge nvidia*
~~~
### 第三步、禁用nouveau驱动（ubuntu自带的显卡驱动）
打开文件blacklist.conf
~~~shell
sudo gedit /etc/modprobe.d/blacklist.conf
~~~
在文件末尾添加
~~~shell
blacklist nouveau
options nouveau modeset=0
~~~
### 第四步、更新系统内核
~~~shell
sudo update-initramfs -u
~~~
重启电脑
### 第五步、检查是否禁用成功
~~~shell
lsmod | grep nouveau
~~~
### 第六步、关闭图形界面（如果原本使用的就是命令行界面忽略此步）
如果不关闭图形界面直接安装，会提示关闭X服务
~~~shell
sudo service lightdm stop
~~~
这时候图形界面会关闭，不要慌，按`Ctrl`-`Alt`+`F1`进入命令行界面，输入用户名和密码登录即可，然后使用正常的命令行操作。
### 第七步、安装驱动
切换到驱动的下载目录
{{% admonition "warning" "注意：" %}}
如果是中文系统，建议将安装包下载至用户目录下，否则将图形见面关闭以后可能会出现中文无法识别的情况
{{% /admonition %}}

~~~shell
cd downloads
sudo sh NVIDIA-Linux-x86_64-430.40.run -no-x-check -no-nouveau-check -no-opengl-files
~~~
参数解释：
~~~
–no-x-check：表示安装驱动时不检查X服务（图形接口服务），如果没有关闭图形界面则必须加上，否则反之。
–no-nouveau-check：表示安装驱动时不检查nouveau驱动，这也是非必需的，因为我们已经在前面步骤中禁用驱动。
–no-opengl-files：表示只安装驱动文件，不安装OpenGL文件。这个参数不可省略，否则会导致登陆界面死循环，英语一般称为”login loop”或者”stuck in login”。 
~~~
{{% admonition "warning" "重点" %}}
–no-opengl-files 这个参数一定要加，否则安装完重启后会登录不了系统，陷入登录死循环。
{{% /admonition %}}

当然，如果忘记加了，一定会出现这种登录死循环的情况，也别慌，这时候在图形登录界面别输入密码，直接使用`Ctrl`-`Alt`+`F1`进入命令行界面，然后按照从第二步再重新来一遍，就可解决。（被无良博主深深残害过）
第八步、重启电脑大功告成
使用`nvidia-smi`查询显卡和驱动信息

##本文严重参考
[Ubuntu安装NVIDA显卡驱动](https://www.cnblogs.com/luofeel/p/8654964.html)


