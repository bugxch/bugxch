# Tensorboad查看pytorch模型


与tensorflow模型与caffe模型不同，当前的pytorch没有官方的直观查看网络结构的工具，google了下pytorch的网络解析的方法，发现可以将pytorch的model转换成为events文件使用tensorboard查看，记录之。

<!--more-->

![](https://pic1.zhimg.com/v2-f5424ffa46911a547362912542c23dd2_1200x500.jpg)


## 安装插件

-  TensorboardX，TensorboardX支持scalar, image, figure, histogram, audio, text, graph, onnx_graph, embedding, pr_curve and videosummaries等不同的可视化展示方式，具体介绍移步至项目Github 观看详情。使用下面的命令安装

    ```python
    pip install tensorboardX
    ```

- 安装tensorboard，参考命令

    ```python
    pip install tensorboard
    ```
## 具体过程

参考代码

```python
#-*-coding:utf-8-*-
import torch
import torchvision
from torch.autograd import Variable
from tensorboardX import SummaryWriter

# 模拟输入数据
input_data = Variable(torch.rand(16, 3, 224, 224))

# 从torchvision中导入已有模型
net = torchvision.models.resnet18()

# 声明writer对象，保存的文件夹
writer = SummaryWriter(log_dir='./log', comment='resnet18')
with writer:
    writer.add_graph(net, (input_data,))
```

该代码中14行声明一个writer对象，分别表示events存放的目录，comment表示事件的title，然后使用如下的方式打开tensorboard

```shell
tensorboard --logpath=D:\log --port=6006
```

然后按照命令行提示打开即可。

## 参考链接

- [PyTorch 使用 TensorboardX 进行网络可视化-PyTorch 中文网](https://www.pytorchtutorial.com/pytorch-tensorboardx/)
- [Pytorch使用tensorboardX可视化。超详细！！！ - 简书](https://www.jianshu.com/p/46eb3004beca)
- [torch.utils.tensorboard — PyTorch master documentation](https://pytorch.org/docs/stable/tensorboard.html)
