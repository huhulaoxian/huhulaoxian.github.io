# Caffe_gpu

GPU版本的caffe安装过程比cpu复杂一些，建议现看cpu版本安装教程，然后再编译gpu版本，会轻松一点。[Caffe(cpu版) Ubuntu安装](https://huhulaoxian.xyz/caffe_cpu/)
<!--more-->

接下来进入正题
## 下载安装nvidia驱动
~~图形界面的ubuntu，直接在设置里的软件和更新里搜索并安装英伟达显卡驱动~~ 

不推荐以上方法安装，ubuntu推荐的显卡驱动并不是最新版，后续安装cuda或其他框架时会对显卡驱动版本有要求，所以推荐使用[这篇文章](https://huhulaoxian.xyz/nvidia_driver/)中的方法正确安装驱动，此处不具体介绍。
## 安装cuda
caffe官方推荐ubuntu16.04安装cuda8.0，英伟达cuda官网下载8.0版本
![/images/caffe_gpu/1.png]
使用 `chmod` 修改文件读写权限
~~~shell
sudo chmod 777 cuda_8.0.44_linux.run
~~~
### 安装
~~~shell
sudo  sh cuda_8.0.44_linux.run
~~~
一路yes，当提示是否安装驱动时，选no，已经安装过了（如果看了[驱动安装](https://huhulaoxian.xyz/nvidia_driver/)的博客）。
### 环境变量配置
打开`~/.bashrc`文件：
~~~shell
sudo gedit ~/.bashrc 
~~~
将以下内容写入到~/.bashrc尾部：
~~~shell
export PATH="/usr/local/cuda-8.0/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda8.0/lib64:$LD_LIBRARY_PATH"
source ~/.bashrc
~~~
### 测试CUDA的samples
~~~shell
1 cd /usr/local/cuda-8.0/samples/1_Utilities/deviceQuery
2 make
3 sudo ./deviceQuery
~~~
## 安装cudnn
caffe官方推荐cudnn(v6)对应cuda8.0版本
下载完成后解压
~~~shell
sudo tar -zxvf ./cudnn-8.0-linux-x64-v6.0.tgz 
~~~
得到一个 cuda 文件夹，该文件夹下`include`和 `lib64` 两个文件夹，命令行进入 `cuda/include` 路径下，然后进行以下操作：
~~~shell
sudo cp cudnn.h /usr/local/cuda/include/ #复制头文件
~~~

然后命令行进入 cudn/lib64 路径下，运行以下命令：
~~~shell
sudo cp lib* /usr/local/cuda/lib64/ #复制动态链接库
sudo ln -sf libcudnn.so.6.0.21 libcudnn.so.6 #生成软衔接
sudo ln -sf libcudnn.so.6 libcudnn.so #生成软链接
~~~
安装完成后可用 `nvcc -V` 命令验证是否安装成功，若出现以下信息则表示安装成功：
![/images/caffe_gpu/1.png]

## 配置makefile.config
将`makefile.config`文件中的以下内容解除注释并按自己的路径配置即可
* 使用cudnn
~~~shell
USE_CUDNN := 1
~~~
* 使用python来写layer
~~~shell
WITH_PYTHON_LAYER := 1
~~~
* 配置python路径
~~~shell
ANACONDA_HOME := $(HOME)/miniconda2
PYTHON_INCLUDE := $(ANACONDA_HOME)/include \
                  $(ANACONDA_HOME)/include/python2.7 \
                  $(ANACONDA_HOME)/lib/python2.7/site-packages/numpy/core/include 
~~~
* 设置hdf5的路径
~~~shell
 INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial 
~~~
* caffe下的Makefile修改地方
  
将：`LIBRARIES += glog gflags protobuf boost_system boost_filesystem m`  
改为：
`LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_serial_hl hdf5_serial`  
将：
`NVCCFLAGS +=-ccbin=$(CXX) -Xcompiler-fPIC $(COMMON_FLAGS)` 
替换为： 
`NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)`

* 编辑`/usr/local/cuda/include/host_config.h`
将其中的第115行注释掉：
~~~shell
#error– unsupported GNU version! gcc versions later than 4.9 are not supported!
~~~
改为
~~~shell
//#error– unsupported GNU version! gcc versions later than 4.9 are not supported!
~~~
## make编译caffe
~~~shell
make all -j8  #j8可以同时使用8个线程编译，加快编译速度
make test
make runtest
~~~
## 编译python接口（使用python调用caffe框架）
~~~shell
make pycaffe
~~~

{{% admonition "warning" "注意" %}}
编译失败重新编译时，需要`make clean`
{{% /admonition %}}

## 导入caffe
* 命令行输入`caffe`，会出现caffe的帮助信息,说明安装成功
* 打开python编辑器，测试caffe的python接口是否可用
~~~
import caffe 
~~~
若报错，找不到模块，需要把caffe下的Python路径导入环境变量中去
~~~shell
vim ~/.bashrc
~~~

{{% admonition "tip" "小贴士" %}}
vim编辑器用不习惯的话，可以使用`gedit`编辑器
{{% /admonition %}}

最后一行加上
~~~
export PYTHONPATH="/home/yaoting/caffe/python:$PYTHONPATH"
~~~
这里的路径写上你自己的路径
然后更新一下`bashrc`
~~~shell
source ~/.bashrc
~~~
## MNIST数据集测试
配置caffe完成后，我们可以利用MNIST数据集对caffe进行测试，过程如下：
1. 将终端定位到Caffe根目录（一定要在caffa的根目录下执行以下命令，否则会出现有些文件找不到的情况）
~~~
cd ~/caffe
~~~
2. 下载MNIST数据库并解压缩
~~~
./data/mnist/get_mnist.sh
~~~
3. 将其转换成Lmdb数据库格式
~~~
./examples/mnist/create_mnist.sh
~~~
4. 训练网络
~~~
./examples/mnist/train_lenet.sh
~~~

## 结尾
Caffe这个深度学习框架上手非常容易，但是安装极难（起码我是这么认为）。较之Tensorflow，Pytorch等框架，caffe组织结构更加清晰，易用性更高，但是存在安装编译困难（我认为如果能装好Caffe，其他框架的安装就是小儿科了）、更新慢（貌似现在已经不更新了，作为老一代深度学习框架，感觉它即将退出历史舞台）、不灵活等缺陷，例如想修改或增加某些新的层，需要修改C++源码，对于新手或没接触过C++的人来说很不友好  

好了，现在可以尝试着将caffe源码中的`example`都跑一下试试看了～
