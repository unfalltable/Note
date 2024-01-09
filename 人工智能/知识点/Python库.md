---
title: Python库
categories: 人工智能
tags: [库]
---

## 文档

Pytorch：https://pytorch.org/docs

## Image

```python
#引入
from PIL import Image

#读取图片，转换为PIL格式
image_path = ""
img = Image.open(image_path);
```

## Numpy 

```python
#引入
import numpy as np
#处理PIL类型的图像
img_array = np.array(i)
```

## Tensorboard

### SummaryWriter

- 用于绘制坐标图

```shell
#下载（pytorch环境下下载）
conda activate pytorch
pip install tensorboard
#可视化界面
tensorboard --logdir=logs --port=6006
```

```python
#引入
from torch.utils.tensorboard import SummaryWriter

#实例化
writer = SummaryWriter("事件文件存储路径")

#添加一张图片
	#tag：标题
	#img_tensor：图像数据类型（torch.Tensor、numpy.array）
	#global_step：步数 / x轴
	#walltime
writer.add_image(tag, img_tensor, global_step, walltime)

#添加多张图片
writer.add_images(tag, img_tensor, global_step, walltime)

#添加标量数据
	#tag：标题
	#scalar_value：数值 / y轴
	#global_step：步数 / x轴
writer.add_scalar(tag, scalar_value, global_step)
#关闭
writer.close()
```

## TransForms

```python
#引入
from torchvision import transforms

#将PIL类型的图片转换为Tensor类型
trans_totensor = transforms.ToTensor()
img_tensor = trans_totensor(img)

#将tensor类型的图片标准化 （-1 ~ 1）
trans_norm = transforms.Normalize([0.5,0.5,0.5],[0.5,0.5,0.5])
#需要传入tensor类型的图片
img_norm = trans_norm(img_tensor)

#Resize图片
#1
trans_resize = transforms.Resize((长,宽))
#传入PIL类型的图片，输出的也是PILle
img_resize = trans_resize(img)

#2
#传入PIL类型的图片，然后resize后输出为tensor类型的图片
trans_resize = transforms.Resize((长,宽))
#相当于把想要执行的操作都作为参数传入，按顺序执行，前一步的返回值必须和后一步的参数匹配，否则报错
trans_compose = transforms.compose(trans_resize, trans_totensor)
#此时输出的就是tensor类型的图片
img_resize = trans_compose(img)

#随机裁剪 RandomCrop
trans_random = transforms.RandomCrop((长,宽))
trans_compose = transforms.compose(trans_random, trans_totensor)
img_random = trans_compose(img)
```

## opencv

```python
#引入
import cv2

#传入图片路径，转化为narrays类型
cv_img = cv2.im
```

## torchvision

```python
#引入
import torchvision

#训练数据集
train_set = torchvision.datasets.CIFAR10(root="./dataset", train=True, download=True)
#测试数据集
test_set = torchvision.datasets.CIFAR10(root="./dataset", train=False, download=True)

#输出的格式为：(图片，识别的结果)
print(train_set[0])
```

## dataloader

```python
#引入
from torch.utils.data import DataLoader

#测试数据集
test_set = torchvision.datasets.CIFAR10(root="./dataset", train=False, download=True)
#loader
	#dataset 数据集
    #batch_size 一次取多少个数据并打包
    #shuffle 是否打乱
    #sampler cai'yang'q
    #batch_sampler
    #num_worker 多进程加载（window下 > 0时可能有问题）
    #drop_last 余数是否舍去
test_loader = DataLoader(dataset=test_set, batch_size=4, shuffle=True, num_worker=0, drop_last=False)
```

## 神经网络框架(torch.nn)

### module 定义

```python
#引入
from torch import nn
import torch

#定义神经网络模板
class Test(nn.module):
    def __init__(self):
        super().__init__()
    #父类中的__call__函数执行了该方法
    def forward(self, input):
        output = input + 1
        return ouput
    
#使用神经网络模板
test = Test()
x = torch.tensor(1.0)
output = test(x)
```

### conv2d 卷积

```python
#引入
import torch
import torch.nn.functional as F

#输入数据
input =  torch.tensor(
	[1,2,3,4,5],
    [1,2,3,4,5],
    [1,2,3,4,5],
    [1,2,3,4,5],
    [1,2,3,4,5])
kernel =  torch.tensor(
	[1,2,3]
    [1,2,3],
    [1,2,3])

#因为输入数据和卷积为二维数组，所以需要升维
	#数据，size, channel, width, high
input = torch.reshape(input, 1, 1, 5, 5)
kernel = torch.reshape(kernel, 1, 1, 3, 3)

#conv2d 
	#input 输入三维类型的数据
    #weight 权重/卷积核
    #bias 偏置
	#stride 步长
    #padding 填充输入数据的长宽
ouput = F.conv2d(input, kernel, stride=1)
```

### maxpool2D 最大池化

```python
#引入
import torch
from torch import nn
from torch.nn import MaxPool2d

#输入数据
input =  torch.tensor(
	[1,2,3,4,5],
    [1,2,3,4,5],
    [1,2,3,4,5],
    [1,2,3,4,5],
    [1,2,3,4,5], dtype=float)
#定义神经网络
class Test(nn.module):
    def __init__(self):
        super(Test, self).__init__()
        self.maxpool1 = MaxPool2d(kernel_size=3, ceil_model=True)

    def forward(self, input):
        output = self.maxpool1(input)
        return ouput
#调用
test = Test()
output = test(input)

#输出
[3, 5]
[3, 5]
```

### 非线性激活

#### ReLU

```python
#引入
import torch
from torch import nn
from torch.nn import ReLU

#输入数据
input =  torch.tensor(
    [[1,-0.5],
    [-1,3]])
input = torch.reshape(input, (-1,1,2,2))

#定义神经网络
class Test(nn.module):
    def __init__(self):
        super(Test, self).__init__()
        self.relu1 = ReLU()
        
    def forward(self, input):
        output = self.relu1(input)
        return ouput
    
#调用
test = Test()
output = test(input)

#输出
[1, 0]
[0, 3]
```

#### Sigmoid

```python
#引入
import torch
from torch import nn
from torch.nn import Sigmoid

#输入数据
input =  torch.tensor(
    [[1,-0.5],
    [-1,3]])
input = torch.reshape(input, (-1,1,2,2))

#定义神经网络
class Test(nn.module):
    def __init__(self):
        super(Test, self).__init__()
        self.sigmoid1 = Sigmoid()
        
    def forward(self, input):
        output = self.sigmoid1(input)
        return ouput
    
#调用
test = Test()
output = test(input)

#输出
[1, 0]
[0, 3]
```

### 线性层

```python
import torch
import torchvision
from torch import nn
from torch.nn import Linearfrom
from torch.utils.data import DataLoader

dataset = torchvision.datasets.CIFAR10(
    	"../data", 
    	train=False,
    	transform=torchvision.transforms,ToTensor(),
    	download=True
)
dataloader = DataLoader(dataset， batch_size=64)

class Tudui(nn.Module):
	def __init__(self):
		super(Tudui，self).__init__()
        self.linear1 = Linear(196608，10)
	def forward(self，input):
        output = self.linear1(input)
        return output
    
tudui_= Tudui()

for data in dataloader:
    imgs， targets = data
    print(imgs.shape)
    #摊平数据
    output = torch.flatten(imgs)
    print(output.shape)
    output = tudui(output)
    print(output.shape)
```

