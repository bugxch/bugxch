# Manjaro安装CUDA教程

去年年底安装将我的Thinkpad T450的双系统中的opensuse换成了[Manjaro](https://manjaro.org/)，折腾安装了下CUDA，是为记录。

<!--more-->

## 基本安装

### NVIDIA显卡安装

Manjaro系统安装显卡比较简单，它有一个命令

```shell
sudo mhwd -a [pci or usb connection] [free or nonfree drivers] 0300
```

其中

- **-a**: 自动检测和安装合适的显卡驱动
- **[pci or usb]**: 为通过PCI或者USB连接的设置安装驱动
- **[free or nonfree]**: 安装免费或者非免费的驱动
- **0300**: 确认即将安装的显卡的驱动

我们要安装英伟达的驱动，只要使用下面的一行命令即可搞定

```shell
sudo mhwd -a pci nonfree 0300
```

等待安装结束，使用如下命令查看是否已经安装完成

```shell
nvidia-smi
```

我的显示结果如下

![](https://github.com/bugxch/blogpics/blob/master/201902/smi.png?raw=true)

从上图可知，我的显卡型号是GeForce 940M，显卡的驱动版本是415.27。

### CUDA安装

#### 安装命令

Manjaro的CUDA安装也是异常简单，一行命令搞定

```shell
sudo pacman -S cuda cudnn
```

这行命令可能需要花费一些时间，请耐心等待。

#### 验证安装

完成之后，我们进入cuda的安装路径，我的路径是`/opt/cuda`，你可以使用下面的命令将CUDA的示例程序拷贝到你的用户主目录下，之后编译程序

```shel
cp -r /opt/cuda/samples ~
cd ~/samples
make
```

此时就使用nvcc编译器开始编译CUDA的sample程序，这个花费时间更长，应该在半小时左右，等待编译结束，使用下面的命令验证是否成功

```shell
cd ~/samples/bin/x86_64/linux/release
./deviceQuery
```

在窗口中查看最后一行的结果是否为pass，如果是则表示CUDA安装成功。

## 双显卡配置

我的电脑有两个显卡，一个是intel的集成显卡，一个是NVIDIA的独显。

### 安装显卡切换程序

Manjaro的双显卡配置有点问题，Bumblebee还是有点问题，使用下面的命令重新安装

```shell
# 依赖
sudo pacman -S virtualgl lib32-virtualgl lib32-primus primus

# 安装双显卡切换程序bumblebee
sudo mhwd -f -i pci video-hybrid-intel-nvidia-bumblebee

# 允许服务
sudo systemctl enable bumblebeed

# 添加用户
sudo gpasswd -a $USER bumblebee
```

为了防止重启之后不能进入登录界面，需要做如下的配置

1. 打开 /etc/default/grub
2. 找到并且改为：GRUB_CMLINE_LINUX_DEFAULT="quiet **acpi_osi=! acpi_osi=Linux acpi_osi=’Windows 2015’ pcie_port_pm=off** resume=..."
3. 运行sudo update-grub，重启

### 测试显卡性能

使用下面的shell命令安装显卡测试程序

```shell
# 安装测试软件
sudo pacman -S mesa-demos

# 集成显卡性能
glxgears -info

# 独显性能
optirun glxgears -info
# 或者
primusrun glxgears -info
```

需要注意的是，之后运行的所有程序，如果需要使用独立显卡，需要在命令的前面加上`optirun`或者`primusrun`的前缀。

```shell
# 打开nvida面板
optirun -b none nvidia-settings -c :8

# 不依赖Bumblebee来使用CUDA
sudo tee /proc/acpi/bbswitch <<< 'ON'

# 使用完CUDA 停止NVIDIA显卡
sudo rmmod nvidia_uvm nvidia && sudo tee /proc/acpi/bbswitch <<< OFF

inxi -G # 查看显卡情况
optirun nvidia-smi # 查看CPU情况
```

## 参考资料

- [Installing Tensorflow 1.6.0 + GPU on Manjaro Linux – Dr. Joe Logan – Medium](https://medium.com/@joelognn/installing-tensorflow-1-6-0-gpu-on-manjaro-linux-9657fa63478)
- [Manjaro折腾笔记：我的数据科学环境搭建之路 - 杨睿 - 博客园](https://www.cnblogs.com/yangruiGB2312/p/9004335.html)
- [leblancfg.com – Notes on installing CUDA, CuDNN and Tensorflow on Manjaro](https://leblancfg.com/installing-cuda-cudnn-tensorflow-nvidia-gtx960.html)
- [Configure Graphics Cards - Manjaro Linux](https://wiki.manjaro.org/index.php/Configure_Graphics_Cards)
