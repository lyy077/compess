---
title: pytorch使用
top: false
cover: false
toc: true
mathjax: true
sticky: 1
date: 2022-04-29 15:09:25
password:
summary:
tags:
categories:
---



### Pytorch

##### 安装

https://pytorch.org/get-started/locally/



#### Pytorch基础

##### tensor的创建与数据类型

1. tensor——张量——是什么？

0阶张量：

```python
import torch


if __name__ == '__main__':
    t0 = torch.tensor(3)
    print(t0.data, t0.shape, t0.size()) # tensor(3) torch.Size([]) torch.Size([])
```

0阶张量就是一个标量，常数。形状为：torch.Size([])，表示没有形状。



1阶张量：

```python
import torch


if __name__ == '__main__':
    t1 = torch.tensor([1, 2, 3, 4, 5, 6, 7, 8, 9])
    print(t1.data, t1.shape, t1.size()) # tensor([1, 2, 3, 4, 5, 6, 7, 8, 9]) torch.Size([9]) torch.Size([9])
```

一阶张量是一个向量vector。

shape是张量的属性，形状属性，因为是一阶张量，所以x轴上的长度为9，表示为torch.Size([9])。size则是张量的方法，返回张量的长度属性。这两个返回的结果显然是一致的。



2阶张量：

```python
import torch


if __name__ == '__main__':
    t2 = torch.empty([3, 4])
    print(t2.data, t2.shape, t2.size()) 
    #  tensor([[9.8091e-45, 0.0000e+00, 0.0000e+00, 0.0000e+00],
    #      [1.4013e-45, 0.0000e+00, 0.0000e+00, 0.0000e+00],
    #      [0.0000e+00, 0.0000e+00, 0.0000e+00, 0.0000e+00]]) torch.Size([3, 4]) torch.Size([3, 4])
```

二阶张量是一个矩阵matrix。除了可以使用torch.tensor去创建张量，还可以用torch.empty创建张量。empty创建的张量数值都是随机的。



2. tensor的创建方法

tensor()	empty()	ones()	zeros()	rand()	randint()	randn()	ones_like()

tensor和empty前面介绍过。



* ones创建的张量数值都是1，ones(3, 4)和ones([3, 4])和ones((3, 4))都表示创建3*4的矩阵。

```python
import torch


if __name__ == '__main__':
    t2 = torch.ones(3, 4)  # torch.ones([3, 4]) torch.ones((3, 4))
    print(t2.data, t2.shape, t2.size())
#  tensor([[1., 1., 1., 1.],
#        [1., 1., 1., 1.],
#        [1., 1., 1., 1.]]) torch.Size([3, 4]) torch.Size([3, 4])
```



* zeros创建张量的用法跟ones一致，只不过创建的张量的数值都是0

```python
import torch


if __name__ == '__main__':
    t2 = torch.zeros(3, 4)  # torch.zeros([3, 4]) torch.zeros((3, 4))
    print(t2.data, t2.shape, t2.size())
#  tensor([[0., 0., 0., 0.],
#        [0., 0., 0., 0.],
#        [0., 0., 0., 0.]]) torch.Size([3, 4]) torch.Size([3, 4])
```

​	

* 虽然跟empty创建张量的数值一样是随机的，但是rand创建的张量的随机值是在[0, 1)这个区间。

```python
import torch


if __name__ == '__main__':
    t2 = torch.rand((3, 4))  # torch.rand([3, 4]) torch.rand((3, 4))
    print(t2.data, t2.shape, t2.size())
#  tensor([[0.9115, 0.4585, 0.7917, 0.0110],
#        [0.6713, 0.1945, 0.4736, 0.3739],
#        [0.2962, 0.8575, 0.7578, 0.2175]]) torch.Size([3, 4]) torch.Size([3, 4])
```



* 创建的张量的值也是随机的，但是却可以指定这个随机数所在的区间。

```python
import torch


if __name__ == '__main__':
    t2 = torch.randint(low=1, high=10, size=[3, 4])  # size=(3, 4)
    print(t2.data, t2.shape, t2.size())
#  tensor([[7, 3, 5, 8],
#        [5, 8, 3, 4],
#        [9, 7, 5, 5]]) torch.Size([3, 4]) torch.Size([3, 4])
```



* randn创建的张量数值虽然是随机的，但是数值符合正态分布。

```python
import torch


if __name__ == '__main__':
    t2 = torch.randn([3, 4])  # torch.randn(3, 4) torch.randn((3, 4]))
    print(t2.data, t2.shape, t2.size())
#  tensor([[ 0.4651, -0.4527,  1.1058,  0.7890],
#        [-0.0802, -0.7622,  0.5029,  0.3051],
#        [ 1.4744,  0.7839, -0.1857, -0.6115]]) torch.Size([3, 4]) torch.Size([3, 4])
```



* ones_like接收一个张量，然后根据提供的张量的形状去构造一个形状一致的张量。

```python
import torch


if __name__ == '__main__':
    t2 = torch.randn([3, 4])
    print(t2.data, t2.shape, t2.size())
    #  tensor([[ 1.3051,  0.8067, -0.6260, -1.1017],
    #    [-1.0816,  1.2157, -0.5902,  1.7605],
    #    [-0.1016,  0.5383, -2.2274, -0.6383]]) torch.Size([3, 4]) torch.Size([3, 4]) 
    t = torch.ones_like(t2)
    print(t, t.shape, t.size())
    #  tensor([[1., 1., 1., 1.],
    #    [1., 1., 1., 1.],
    #    [1., 1., 1., 1.]]) torch.Size([3, 4]) torch.Size([3, 4])
```

注意，只是形状一致，创建的张量的数值依旧是在[0, 1)这个区间。



3. tensor的数据类型

获取tensor的数据类型：tensor.dtype

```python
torch.float == torch.float32
torch.double == torch.float64
torch.int == torch.int32
torch.long == torch.int64
torch.short = torch.int16
torch.float16
torch.uint8
torch.bool
```



看下前面介绍的几种创建张量的方法创建出来的张量数值类型默认是什么？

```python
import torch


if __name__ == '__main__':
    t1 = torch.tensor(1)
    print(t1.dtype)	 # torch.int64
    t2 = torch.tensor([3, 4])
    print(t2.dtype)  # torch.int64
    t3 = torch.empty(3, 4)
    print(t3.dtype)  # torch.float32
    t4 = torch.ones(3, 4)
    print(t4.dtype)	 # torch.float32
    t5 = torch.rand(3, 4)
    print(t5.dtype)  # torch.float32
    t6 = torch.randn(3, 4)
    print(t6.dtype)  # torch.float32
    t7 = torch.randint(low=1, high=10, size=(3, 4))
    print(t7.dtype)  # torch.float32
```



在创建张量的时候，也可以指定数值的数据类型：

```python
import torch


if __name__ == '__main__':
    t1 = torch.tensor(0, dtype=torch.bool)
    print(t1, t1.dtype, t1.shape)  # tensor(False) torch.bool torch.Size([])
```



也可以使用封装好的创建指定类型数值的张量方法：

```python
import torch


if __name__ == '__main__':
    t1 = torch.DoubleTensor(3, 4)
    print(t1, t1.dtype, t1.shape)
    # tensor([[ 0.0000e+00, 1.2599e-321,  0.0000e+00,  0.0000e+00],
    #    [ 0.0000e+00,  0.0000e+00, 1.2599e-321,  0.0000e+00],
    #    [ 0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00]],
    #   dtype=torch.float64) torch.float64 torch.Size([3, 4])
```

其它的还有，torch.Tensor, torch.FloatTensor, torch.BoolTensor, torch.IntTensor, torch.LongTensor, torch.ShortTensor, torch.ByteTensor, torch.CharTensor。



##### tensor的方法

1. 获取数据的形状

shape属性或者size()，如果想得到矩阵的行或者列，该如何操作：

```python
import torch


if __name__ == '__main__':
    t2 = torch.IntTensor(3, 4)
    print(t2.shape[0], t2.size()[1])  # 3 4 t2.size()[1]也可以写成t2.size(1)
```

shape属性和size()方法返回的是一个列表，直接取就行。



2. 获取数据类型

dtype



3. 数据类型的转换

tensor.类型：

```python
import torch


if __name__ == '__main__':
    t2 = torch.IntTensor(3, 4)
    print(t2, t2.dtype)
    #  tensor([[1, 0, 0, 0],
    #    [1, 0, 0, 0],
    #    [0, 0, 0, 0]], dtype=torch.int32) torch.int32
    t2 = t2.long()
    print(t2, t2.dtype)
    #  tensor([[1, 0, 0, 0],
    #    [1, 0, 0, 0],
    #    [0, 0, 0, 0]]) torch.int64
```



4. 单个tensor的快速取值

tensor.item()

```python
import torch


if __name__ == '__main__':
    t = torch.tensor([[[[[100]]]]])
    print(t, t.dtype, t.shape, t[0][0][0][0][0].numpy())  # tensor([[[[[100]]]]]) torch.int64 torch.Size([1, 1, 1, 1, 1]) 100
```

考虑上面一个张量，如果想快速取数值100，得像`t[0][0][0][0][0]`去调用，而且这个返回的是`tensor(100)`这种数据类型，并不是Python的数值类型，还需要借助tensor转numpy的numpy()方法才行。这样显得很麻烦。



有没有简单点的方式，使用item()快速取值：

```python
import torch


if __name__ == '__main__':
    t = torch.tensor([[[[[100]]]]])
    print(t.item())  # 100

    t1 = torch.tensor([[[[[3, 4]]]]])
    print(t1, t1.shape, t1.dtype)  # tensor([[[[[3, 4]]]]]) torch.Size([1, 1, 1, 1, 2]) torch.int64 
    print(t1.item())  # ValueError: only one element tensors can be converted to Python scalars
```

但是item有一个问题，就是item取值，只针对张量里面只有一个数值的情况，如果有多个就不行，会报错。



5. tensor转numpy

tensor.numpy()，如上边介绍的。



6. 改变形状

tensor.view()

```python
import torch


if __name__ == '__main__':
    t = torch.rand(3, 4, 2)
    print(t)
    #  tensor([[[0.5738, 0.4219],
    #     [0.9189, 0.8043],
    #     [0.1150, 0.1686],
    #     [0.4825, 0.9803]],

    #    [[0.9246, 0.3613],
    #     [0.5014, 0.7413],
    #     [0.5580, 0.0173],
    #     [0.3500, 0.9958]],

    #    [[0.3011, 0.7859],
    #     [0.0362, 0.2246],
    #     [0.2623, 0.1635],
    #     [0.5559, 0.9756]]])
    print(t.view(-1))
    #  tensor([0.5738, 0.4219, 0.9189, 0.8043, 0.1150, 0.1686, 0.4825, 0.9803, 0.9246,
    #    0.3613, 0.5014, 0.7413, 0.5580, 0.0173, 0.3500, 0.9958, 0.3011, 0.7859,
    #    0.0362, 0.2246, 0.2623, 0.1635, 0.5559, 0.9756])
    print(t.view(-1, 2))
    #  tensor([[0.1306, 0.5364],
    #    [0.4956, 0.9338],
    #    [0.9695, 0.5630],
    #    [0.1422, 0.7782],
    #    [0.2201, 0.2908],
    #    [0.2542, 0.2665],
    #    [0.3879, 0.5353],
    #    [0.0152, 0.9092],
    #    [0.7311, 0.7608],
    #    [0.3126, 0.1075],
    #    [0.3832, 0.4845],
    #    [0.8370, 0.5387]])
```



7. 获取阶数

tensor.dim()

```python
import torch


if __name__ == '__main__':
    t = torch.rand(3, 4, 2)
    print(t.dim())  # 3
```



8. 求值

tensor.max(), tensor.min(), tensor.mean(), tensor.std(), tensor.sqrt(), tensor.pow(2), tensor.sum()



* 先看看max(min同理)

```python
import torch


if __name__ == '__main__':
    t = torch.rand(3, 4, 2)
    print(t)
    # tensor([[[0.2396, 0.9697],
    #          [0.7198, 0.4427],
    #          [0.9324, 0.8745],
    #          [0.6353, 0.1289]],
    # 
    #         [[0.9161, 0.9783],
    #          [0.3430, 0.0946],
    #          [0.9589, 0.8446],
    #          [0.6192, 0.1667]],
    # 
    #         [[0.7493, 0.2932],
    #          [0.1378, 0.5082],
    #          [0.9469, 0.0988],
    #          [0.2633, 0.7222]]])
    print(t.max(dim=-1))
    # torch.return_types.max(
    #     values=tensor([[0.9697, 0.7198, 0.9324, 0.6353],
    #                    [0.9783, 0.3430, 0.9589, 0.6192],
    #                    [0.7493, 0.5082, 0.9469, 0.7222]]),
    #     indices=tensor([[1, 0, 0, 0],
    #                     [1, 0, 0, 0],
    #                     [0, 1, 0, 1]]))

```

max(dim=-1)，表示最后一个维度求max，最后一个维度就是最里面的这个列表，比如`[0.2396, 0.9697]`。我们看看计算出来的结果包含2部分，第一部分是值，第二部分是索引。`[0.2396, 0.9697]`最大值是0.9697，对应索引为1，所以values第一个元素的值是0.9697，indices第一个元素的值是1，依次类推。然后看下最终得到的结果变成了2维，从3维降为2维，因为最里面的那个维度取最值之后就变成了一个数值。



还可以把values和indices分开取：

```python
value, indices = t.max(dim=-1)
print(value)
# tensor([[0.4141, 0.8909, 0.8394, 0.2918],
#        [0.2939, 0.3287, 0.7192, 0.9056],
#        [0.2104, 0.7899, 0.6259, 0.4323]])
print(indices)
# tensor([[0, 1, 0, 0],
#        [0, 0, 0, 1],
#        [0, 0, 0, 0]])
```



* meam

mean表示求均值，在求均值之前必须把张量转为float类型：

```python
import torch


if __name__ == '__main__':
    t1 = torch.tensor([1, 2, 3, 4, 5, 6, 7, 8, 9])
    print(t1.float().mean()) 			  # tensor(5.)
    print(t1.float().sum() / t1.shape[0]) # tensor(5.)
    print(t1.float().sum() / len(t1))	  # tensor(5.)
```



这里说下len(tensor)表示的含义：理解成多维数组求长度。

```python
import torch


if __name__ == '__main__':
    t1 = torch.tensor([1, 2, 3, 4, 5, 6, 7, 8, 9])
    print(len(t1))	# 3
    t2 = torch.rand([2, 3])
    print(len(t2))	# 2
    t3 = torch.rand(2, 3, 4)
    print(len(t3))	# 2
    t4 = torch.rand(3, 2, 1)
    print(len(t4))	# 9
```

可以这样理解：t1表示一个一维数组，所以长度就为这个数组的长度，即9。

​						   t2表示一个数组，这个数组有2个元素，里面每一个元素都是一个一维数组。所以len(t2)自然是2

​							...

​							依次类推。



* std表示标准差。

```python
import torch
import math


if __name__ == '__main__':
    t1 = torch.tensor([1, 2, 3, 4, 5, 6, 7, 8, 9])
    print(t1.float().std()) 
    # tensor(2.7386)
    print(torch.sqrt((t1.float() - t1.float().mean()).pow(2).sum() / (t1.size(0) - 1))) 
    #tensor(2.7386)
    print(math.sqrt(sum(list(map(lambda x: math.pow(x.float() - t1.float().mean(), 2), t1))) / (t1.size(0) - 1)))
    # 2.7386127875258306
```

第一种是使用std方法，第二种是使用tensor的相关api，第三种使用python math库的一些方法。

要注意的是t1.float() - 5表示对所有维度上的值都减去5，如下：

```python
import torch


if __name__ == '__main__':
    t1 = torch.randint(low=1, high=5, size=(2, 3, 4))
    print(t1)
    # tensor([[[3, 1, 1, 1],
    #     [1, 3, 3, 4],
    #     [3, 2, 4, 2]],

    #    [[3, 2, 3, 4],
    #     [2, 2, 4, 3],
    #     [4, 2, 1, 2]]])
    print(t1 - 5)
	# tensor([[[-2, -4, -4, -4],
    #     [-4, -2, -2, -1],
    #     [-2, -3, -1, -3]],

    #    [[-2, -3, -2, -1],
    #     [-3, -3, -1, -2],
    #     [-1, -3, -4, -3]]])
```



9. 带下划线的方法：就地修改。

```python
t1 = tensor([1, 2, 3, 4, 5])
t1 = t1.add(5)
t1.add_(5) # 跟上面的等价
```



10. 轴交换

permute表示轴交换，即维度交换。

```python
import torch


if __name__ == '__main__':
    t1 = torch.randint(low=1, high=5, size=(2, 3, 4))
    print(t1.size()) # torch.Size([2, 3, 4])
    t1 = t1.permute([2, 0, 1])
    print(t1.size()) # torch.Size([4, 2, 3])
```

比如有一个三维的tensor，其形状为x轴为2，y轴为3，z轴为4。如果想变成x轴为4，y轴为2，z轴为3。则需要使用permute函数，x轴要变成4，则可以和原来的z轴交换，原来的z轴索引为2，则第一个参数传2；同理，第二个参数传0，第三个参数传1。



permute和view的区别在于，permute是表示轴交换，例如把x轴的数据和y轴的数据交换位置，而view表示改变形状，比如先把整个形状拍扁，然后再根据要求分组重组形状。



##### 使用GPU进行运算

判断cuda是否可用：`torch.cuda.is_available()`

创建一个cuda设备：`torch.device('cuda')`

将数据推送到coda上去运算：`tensor.to(device)`

推送数据同时转换类型：`tensor.to('cpu', torch.double)`

```python
import torch


if __name__ == '__main__':
    t1 = torch.tensor([1, 2, 3, 4, 5])
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    t1.to(device)
    print(t1)
```



##### 数据加载

1. 环境安装

```shell
pip install tqdm          # 进度条
pip install matplotlib    # 展示图片
pip install opencv-python # cv2       
```



2. 深度学习图像识别的基本步骤

* 准备数据集
* 模型的构建
* 模型的训练
* 模型的保存
* 模型的评估



3. 数据集分类：Pytorch自带的数据集，自定义数据集

4. 数据集类与数据加载类：

    torch.utils.data.Dataset:

    ​	继承Dataset类后，必须要实现的2个方法：\_\_getitem\_\_和\_\_len\_\_。

    torch.utils.data.DataLoader：

    ​	

5. MNIST相关参数介绍：

    MNIST是pytorch自带的一个数据集类，继承自torch.utils.data.Dataset。

    其中，包含训练集60000张，测试集10000张，图片大小28\*28\*1

    区分几个概念：训练集，验证集，测试集，MNIST把训练集和测试集归为一类。

    ```python
    from torchvision.datasets import MNIST
    
    if __name__ == '__main__':
        mnist1 = MNIST(root="./data", train=True, download=True, transform=None)
    ```

​		MNIST第一个参数root传一个目录，表示数据集下载后存放的目录。download表示是否下载，如果指定的目录不为空，不会重复下载，train为true表示训练集，否则为测试集。



可以打印下训练集和测试集的图片张数：

```python
from torchvision.datasets import MNIST

if __name__ == '__main__':
    mnist1 = MNIST("./data", train=True, download=True, transform=None)
    mnist2 = MNIST("./data", train=False, download=False, transform=None)
    print(len(mnist1))
    print(len(mnist2))
```

结果如下：

![image-20220429224251837](https://img.heshipeng.com/202204292242309.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



6. DataLoader相关参数介绍

```
from torch.utils.data import DataLoader
from torchvision.datasets import MNIST

if __name__ == '__main__':
    mnist1 = MNIST("./data", train=True, download=True, transform=None)
    DataLoader(mnist1, batch_size=16, shuffle=True)
```



看下DataLoader的构造函数的几个重要参数：

![image-20220429225325231](https://img.heshipeng.com/202204292253333.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



dataset表示数据集，这里传入之前实例化好的mnist1

batch_size表示一次加载的图片数量，这里默认为1，表示一张。

shuffle表示加载数据集的时候是否打乱，一般设置为True表示打乱

num_workers表示加载数据集的进程数量，根据服务器的配置去调整。

drop_last表示如果最后一批图片数量不足batch_size的大小，是否丢掉，为True表示丢掉。



下面一段代码用来展示MNIST数据集的一个batch_size大小的图片：

```python
import matplotlib.pyplot as plt
import torchvision.utils
from torch.utils.data import DataLoader
from torchvision import transforms
from torchvision.datasets import MNIST


def im_show(inp):
    inp = inp.numpy().transpose((1, 2, 0))
    # 或者 inp = inp.permute([1, 2, 0])
    # 或者 inp = transforms.ToPILImage()(inp)
    plt.imshow(inp)
    plt.pause(10)


if __name__ == '__main__':
    mnist1 = MNIST("./data", train=True, download=True, transform=transforms.ToTensor())
    data_loader = DataLoader(mnist1, batch_size=16, shuffle=True)
    print(mnist1[0])

    for images, labels in data_loader:
        print(images.shape)
        print(labels.shape)
        print(labels)
        out = torchvision.utils.make_grid(images)
        im_show(out)
        break

```

transform=transforms.ToTensor()这里调用ToTensor方法把PIL文件转为Tensor对象，DataLoader返回的是一个可迭代对象，每一次迭代返回一个元组，第一个元素是图片，第二个数据是label。make_grid方法把若干张图片打包成网格形式，方便查看。im_show调用matplotlib相关的api去展示图片，图片输出如下：

![image-20220429232932811](https://img.heshipeng.com/202204292329977.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



看下程序的输出：

![image-20220429233012655](/Users/bird/Library/Application%20Support/typora-user-images/image-20220429233012655.png)

第一行表示image的shape，16表示batch_size为16，1表示通道数，因为是黑白图，所以是单通道，值为1，后边2个28表示高和宽。

第二行表示label的shape，16表示batch_size为16

第三行表示label的值，5，6，2，4...分别对应图片中的数字正确的结果，用来和预测出来的结果做比较。



7. torchvision.utils.make_grid图片打包成网格来显示。

用法如上。



##### 图像处理

看看上节的`mnist1 = MNIST("./data", train=True, download=True, transform=transforms.ToTensor())`做了什么？

如果transform不设置成transform.ToTensor()，那么得到的MNIST就会是一个PIL对象，如下：

![image-20220430112937360](https://img.heshipeng.com/202204301129495.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



但是dataloader需要接收的是tensor对象。所以设置transform=transforms.ToTensor()把PIL转为tensor。



transforms.ToTensor具体做了什么事呢？

一般读取进来的图片是h * w * c的，Pytorch无法处理。ToTensor把图片数据处理成c * h * w的，这样Pytorch才能处理，但是要用到其它的库去展示图片的时候，再还原处理成h * w * c。

所以上边代码有一行`inp = inp.numpy().transpose((1, 2, 0))`就是这个意思。轴交换也可以用我们之前讲过的permute方法，所以代码也可以替换成：`inp = inp.permute([1, 2, 0])`，还可以直接使用tensor转PIL的方法：`inp = transforms.ToPILImage()(inp)`这样也是可以。



图像增强：指的是对图片进行缩放，拉伸，选装，标准化处理一类的操作。相当于增加样本的数量，减少过拟合，提升模型泛化能力。



transform常用的一些api：

```python
from torchvision import transforms


if __name__ == '__main__':
    # 打包
    transform = transforms.Compose([
        # 转化为PIL图片
        transforms.ToPILImage(),
        # 缩放到目标大小
        transforms.Resize(size),
        # 转换为张量
        transforms.ToTensor(),
        # 标准化处理
        transforms.Normalize(
            mean=(0.1307,),
            std=(0.3081,)
        )
    ])

    # transforms.RandomRotation 随机旋转
    # transforms.RandomHorizontalFlip 随机水平翻转
    # transforms.RandomVerticalFlip 随机垂直翻转
    # transforms.RandomResizedCrop 随机截取一部分

```



#### 神经网络

##### 全连接层

![image-20220430175953615](https://img.heshipeng.com/202204301759708.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



黄色的：输入，x1 x2 x3 x4理解成输入的四个像素点

中间的蓝色叫做全连接层，一共有5层全连接层



![image-20220502171741332](https://img.heshipeng.com/202205021717460.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



![image-20220502171930092](https://img.heshipeng.com/202205021719186.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### OneHot编码





#### 损失函数

比较重要：数据集，网络模型，损失函数



1. 均方误差，一般用来做回归任务

nn.MSELoss()

torch.nn.functional.mse_loss(input, target)



2. 交叉熵损失 

nn.CrossEntrypyLoss()



import torch.nn.functional as F

new_out = F.log_softmax(out, dim=1)

F.nll_loss(new_out, label)





#### 交叉熵损失





#### 模型的训练





#### 模型的评估





#### 卷积神经网络

![image-20220503104053936](https://img.heshipeng.com/202205031040089.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



卷积过程，特征值计算，参数共享，卷积的参数量计算，卷积核 kernal_size，步长 stride，边缘填充 padding
