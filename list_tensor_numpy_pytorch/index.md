# Pytorch中List、tensor、numpy相互转换

本文介绍了pytorch中list、numpy、tensor之间的互相转化
<!--more-->
### list 转 numpy
~~~
ndarray = np.array(list)
~~~
### numpy 转 list
~~~
list = ndarray.tolist()
~~~
### list 转 torch.Tensor
~~~
tensor=torch.Tensor(list)
~~~
### torch.Tensor 转 list
~~~
# 先转numpy，后转list
list = tensor.numpy().tolist()
~~~
### torch.Tensor 转 numpy
~~~
ndarray = tensor.numpy()
# *gpu上的tensor不能直接转为numpy,应先放回cpu中
ndarray = tensor.cpu().numpy()
~~~
### numpy 转 torch.Tensor
~~~
tensor = torch.from_numpy(ndarray) 
~~~
