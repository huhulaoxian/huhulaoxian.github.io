# Pytorch中图像格式的互相转换

做计算机视觉避免不了的就是图片与张量之间的转换，本文介绍了PIL图像与Pytorch中tensor，以及numpy所读取的opencv图像之间的互相转化
<!--more-->
## 预备知识
我们一般在pytorch或者python中处理的图像无非这几种格式：
* PIL.IMAGE：使用python自带图像处理库读取出来的图片格式
* numpy：使用python-opencv库读取出来的图片格式
* tensor：pytorch中训练时所采取的向量格式（当然也可以说图片）
注意，之后的讲解图片格式皆为RGB三通道，24-bit真彩色，也就是我们平常使用的图片形式
## PIL与Tensor转换
PIL.IMAGE是Pytorch官方推荐的图片处理工具，所以理应对他们之间的转换更加方便一些，官方也写好了函数方便转换，调用即可，开始转换之前当然要把他们的包先导入
~~~python
import torch
from PIL import Image
import torchvision.transforms as transforms
~~~
先来看看这两个互相转换的函数是什么
~~~python
# PIL转Tensor
loader = transforms.Compose([
    transforms.ToTensor()])  #Compose函数是一个形变集合，可以将多个transforms放到一起，顺序执行
# 当然你也可以这样写
loader = transforms.ToTensor()
# Tensor转PIL
unloader = transforms.ToPILImage()
~~~
### PIL转Tensor
1. 使用PIL读取图像
2. 转换为Tensor
~~~python
image = Image.open(image_url).convert('RGB')
image = loader(image)
# 至此图像已经被转换为张量（tensor）
image.size()
# 输出为‘torch.Size([通道数, 高, 宽])’
~~~
### Tensor转为PIL图片
1. 输入tensor变量
2. 输出PIL格式图片
~~~python
image = unloader(image)
image.size  # 注意PIL的.size函数没有括号
# 输出‘(宽, 高)’，注意这里的宽高的顺序与tensor相反
~~~
## Numpy与Tensor
使用cv2读取出来的图片通常是Numpy格式，需要注意的是用python-opencv读取出来的图片和使用PIL读取出来的图片数据略微不同，主要体现在他的通道排列为BGR，而非主流的RGB，形状为（高,宽,通道数）
还是先引入包
~~~python
import cv2
import torch
import numpy as np
~~~
### numpy转化为tensor
先来看一下cv读取图片后变为np矩阵的形状以及图片转为Tensor后的形状
np：（高，宽，通道数）
tensor：（通道数, 高, 宽）
还要注意cv的默认通道排序是BGR，而非主流的RGB，需要做一个转换，知道这个，下面的代码就好理解了
~~~python
img = cv2.imread(image_url)
img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)  # COLOR_BGR2RGB 表示转换通道顺序
img = torch.from_numpy(img.transpose((2, 0, 1)))    # transpose((2, 0, 1)) 表示将（高，宽，通道数）转换为（通道数, 高, 宽）
img = img.float().div(255).unsqueeze(0)    # div表示除255，做归一化
#至此np格式的opencv图像便转换为tensor
~~~
### tensor转Numpy
~~~python
img = tensor.mul(255).byte()    #mul(255) 表示乘255
img = img.cpu().numpy().squeeze(0).transpose((1, 2, 0))     # transpose((1, 2, 0)还原顺序
~~~
## PIL与Numpy
PIL.image.size:（宽，高）
np.shape:（高，宽，通道数）
### PIL转Numpy
~~~python
img = Image.open(image_url)
np = np.array(img)
~~~
### Numpy转PIL
~~~python
img = Image.fromarray(np.uint8(np))
~~~
