# matplot 之 3D 绘图指南


因为梯度下降算法需要绘制 3 维图像，故学习之，日后借鉴。

<!--more-->

![](https://github.com/bugxch/blogpics/blob/master/201807/3d_demo.png?raw=true)

本文稿翻译自 [mplot3d tutorial — Matplotlib 2.0.2 documentation](https://matplotlib.org/mpl_toolkits/mplot3d/tutorial.html)。

使用 matplotlib 绘制 3D 图像，一般要加入一个新的 axes 类型 Axes3D:

```
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')
```

其中的`ax`，就是添加一个三维坐标系`Axes3D`的对象，如下图所示

![](https://github.com/bugxch/blogpics/blob/master/201807/3dax.png?raw=true)
3D 图形分为如下几类：

## 线形图

`Axes3D.plot(xs,ys,**args,**kwargs)`

绘制 2D 或者 3D 的数据。

| Argument   | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| *xs*, *ys* | x, y coordinates of vertices                                 |
| *zs*       | z value(s), either one for all points or one for each point. |
| *zdir*     | Which direction to use as z (‘x’, ‘y’ or ‘z’) when plotting a 2D set. |

关键参数传给了`plot()`函数，例如下面的代码

```python
import matplotlib as mpl
from mpl_toolkits.mplot3d import Axes3D
import numpy as np
import matplotlib.pyplot as plt

mpl.rcParams['legend.fontsize'] = 10

fig = plt.figure()
ax = fig.gca(projection='3d')
theta = np.linspace(-4 * np.pi, 4 * np.pi, 100)
z = np.linspace(-2, 2, 100)
r = z**2 + 1
x = r * np.sin(theta)
y = r * np.cos(theta)
ax.plot(x, y, z, label='parametric curve')
ax.legend()

plt.show()
```

绘制的图形如下

![line3d](https://matplotlib.org/mpl_examples/mplot3d/lines3d_demo.png)

从这个例子可以看出，matplot 画图的基本步骤包括：导入必要的模块，创建 figure 对象，设置 3D 的 ax，创建自变量，写出函数关系式，绘制图形。

## 散点图

`Axes3D.scatter(*xs*, *ys*, *zs=0*, *zdir='z'*, *s=20*, *c=None*, *depthshade=True*, **args*, **\*kwargs*)`

| Argument     | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| *xs*, *ys*   | Positions of data points.                                    |
| *zs*         | Either an array of the same length as *xs* and *ys* or a single value to place all points in the same plane. Default is 0. |
| *zdir*       | Which direction to use as z (‘x’, ‘y’ or ‘z’) when plotting a 2D set. |
| *s*          | Size in points^2. It is a scalar or an array of the same length as *x* and *y*. |
| *c*          | A color. *c* can be a single color format string, or a sequence of color specifications of length *N*, or a sequence of *N*numbers to be mapped to colors using the *cmap* and *norm* specified via kwargs (see below). Note that *c* should not be a single numeric RGB or RGBA sequence because that is indistinguishable from an array of values to be colormapped. *c* can be a 2-D array in which the rows are RGB or RGBA, however, including the case of a single row to specify the same color for all points. |
| *depthshade* | Whether or not to shade the scatter markers to give the appearance of depth. Default is *True*. |

关键参数传给了`scatter()`函数，如下面的例子

```python
'''
==============
3D scatterplot
==============

Demonstration of a basic scatterplot in 3D.
'''

from mpl_toolkits.mplot3d import Axes3D
import matplotlib.pyplot as plt
import numpy as np


def randrange(n, vmin, vmax):
    '''
    Helper function to make an array of random numbers having shape (n, )
    with each number distributed Uniform(vmin, vmax).
    '''
    return (vmax - vmin)*np.random.rand(n) + vmin

fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')

n = 100

# For each set of style and range settings, plot n random points in the box
# defined by x in [23, 32], y in [0, 100], z in [zlow, zhigh].
for c, m, zlow, zhigh in [('r', 'o', -50, -25), ('b', '^', -30, -5)]:
    xs = randrange(n, 23, 32)
    ys = randrange(n, 0, 100)
    zs = randrange(n, zlow, zhigh)
    ax.scatter(xs, ys, zs, c=c, marker=m)

ax.set_xlabel('X Label')
ax.set_ylabel('Y Label')
ax.set_zlabel('Z Label')

plt.show()
```

这个函数里画了 2 组散点图，分别用其中的小三角和红色的圆点表示。函数`randrange`产生`[vmin,vmax]`上的均匀分布的一列数。如下图所示

![scatter](https://matplotlib.org/mpl_examples/mplot3d/scatter3d_demo.png)

## 线框图

`Axes3D.plot_wireframe(X, Y, Z, args, kwargs)`

绘制 3D 的线框图，其中的参数`rstride`和`cstride`表示对输入数据的采样，它们不能和`rcount`以及`ccount`同时使用，不然会产生错误，后者表示从输入数据中采样多少以生成线框图。

| Argument  | Description                                    |
| --------- | ---------------------------------------------- |
| *X*, *Y*, | Data values as 2D arrays                       |
| *Z*       |                                                |
| *rstride* | Array row stride (step size), defaults to 1    |
| *cstride* | Array column stride (step size), defaults to 1 |
| *rcount*  | Use at most this many rows, defaults to 50     |
| *ccount*  | Use at most this many columns, defaults to 50  |

关键参数传给了`Linecollection`，返回一个`Line3DCollection`的类。举例如下

```python
'''
=================
3D wireframe plot
=================

A very basic demonstration of a wireframe plot.
'''

from mpl_toolkits.mplot3d import axes3d
import matplotlib.pyplot as plt


fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')

# Grab some test data.
X, Y, Z = axes3d.get_test_data(0.05)

# Plot a basic wireframe.
ax.plot_wireframe(X, Y, Z, rstride=10, cstride=10)

plt.show()
```

绘制图形如下

![frame](https://matplotlib.org/mpl_examples/mplot3d/wire3d_demo.png)

其中`rstride`和`cstride`分别代表采样的密度，这里是每隔 10 个点计算一个 Z 值，如果设置成 1，绘制的图形会更密集，如下图所示

![](https://github.com/bugxch/blogpics/blob/master/201807/framce.png?raw=true)

## 表面图

`Axes3D.plot_surface(X, Y, Z, *args, **kwargs)`

默认使用纯色为阴影着色，不过它也可以通过 *cmap* 支持颜色映射。

| Argument      | Description                                      |
| ------------- | ------------------------------------------------ |
| *X*, *Y*, *Z* | Data values as 2D arrays                         |
| *rstride*     | Array row stride (step size)                     |
| *cstride*     | Array column stride (step size)                  |
| *rcount*      | Use at most this many rows, defaults to 50       |
| *ccount*      | Use at most this many columns, defaults to 50    |
| *color*       | Color of the surface patches                     |
| *cmap*        | A colormap for the surface patches.              |
| *facecolors*  | Face colors for the individual patches           |
| *norm*        | An instance of Normalize to map values to colors |
| *vmin*        | Minimum value to map                             |
| *vmax*        | Maximum value to map                             |
| *shade*       | Whether to shade the facecolors                  |

其他的参数传给`Ploy3DCollection`，举例如下

```python
'''
======================
3D surface (color map)
======================

Demonstrates plotting a 3D surface colored with the coolwarm color map.
The surface is made opaque by using antialiased=False.

Also demonstrates using the LinearLocator and custom formatting for the
z axis tick labels.
'''

from mpl_toolkits.mplot3d import Axes3D
import matplotlib.pyplot as plt
from matplotlib import cm
from matplotlib.ticker import LinearLocator, FormatStrFormatter
import numpy as np


fig = plt.figure()
ax = fig.gca(projection='3d')

# Make data.
X = np.arange(-5, 5, 0.25)
Y = np.arange(-5, 5, 0.25)
X, Y = np.meshgrid(X, Y)
R = np.sqrt(X**2 + Y**2)
Z = np.sin(R)

# Plot the surface.
surf = ax.plot_surface(X, Y, Z, cmap=cm.coolwarm,
                       linewidth=0, antialiased=False)

# Customize the z axis.
ax.set_zlim(-1.01, 1.01)
ax.zaxis.set_major_locator(LinearLocator(10))
ax.zaxis.set_major_formatter(FormatStrFormatter('%.02f'))

# Add a color bar which maps values to colors.
fig.colorbar(surf, shrink=0.5, aspect=5)

plt.show()
```

绘制图形如下

![plot3d](https://matplotlib.org/mpl_examples/mplot3d/surface3d_demo.png)

参考图形知道`cm`用来做 color mapping，重新设置`arange`的步长为 0.01，可以得到如下的图形

![](https://github.com/bugxch/blogpics/blob/master/201807/cr.png?raw=true)

表面光滑细致多了。

## 2D/3D 图形共存

这篇文章主要是用来画 3 维图形的，以上的几个图形已经够用，下面介绍一些其他的技能。现在的是在 2D 中画 3D 图形。直接上代码和图像

```python
"""
=======================
Plot 2D data on 3D plot
=======================

Demonstrates using ax.plot's zdir keyword to plot 2D data on
selective axes of a 3D plot.
"""

from mpl_toolkits.mplot3d import Axes3D
import numpy as np
import matplotlib.pyplot as plt

fig = plt.figure()
ax = fig.gca(projection='3d')

# Plot a sin curve using the x and y axes.
x = np.linspace(0, 1, 100)
y = np.sin(x * 2 * np.pi) / 2 + 0.5
ax.plot(x, y, zs=0, zdir='z', label='curve in (x,y)')

# Plot scatterplot data (20 2D points per colour) on the x and z axes.
colors = ('r', 'g', 'b', 'k')
x = np.random.sample(20*len(colors))
y = np.random.sample(20*len(colors))
c_list = []
for c in colors:
    c_list.append([c]*20)
# By using zdir='y', the y value of these points is fixed to the zs value 0
# and the (x,y) points are plotted on the x and z axes.
ax.scatter(x, y, zs=0, zdir='y', c=c_list, label='points in (x,z)')

# Make legend, set axes limits and labels
ax.legend()
ax.set_xlim(0, 1)
ax.set_ylim(0, 1)
ax.set_zlim(0, 1)
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')

# Customize the view angle so it's easier to see that the scatter points lie
# on the plane y=0
ax.view_init(elev=20., azim=-35)

plt.show()
```

![2d/3d](https://matplotlib.org/_images/2dcollections3d_demo.png)

从上面的代码，可以看出如何设置坐标轴的取值范围，设置 label 的方法。

## 加入文字

`Axes3D.text(x, y, z, s, zdir=None, **kwargs)`

在画图中我们可能需要在特定位置加入文字说明，下面就是一个例子

```python
'''
======================
Text annotations in 3D
======================

Demonstrates the placement of text annotations on a 3D plot.

Functionality shown:
- Using the text function with three types of 'zdir' values: None,
  an axis name (ex. 'x'), or a direction tuple (ex. (1, 1, 0)).
- Using the text function with the color keyword.
- Using the text2D function to place text on a fixed position on the ax object.
'''

from mpl_toolkits.mplot3d import Axes3D
import matplotlib.pyplot as plt


fig = plt.figure()
ax = fig.gca(projection='3d')

# Demo 1: zdir
zdirs = (None, 'x', 'y', 'z', (1, 1, 0), (1, 1, 1))
xs = (1, 4, 4, 9, 4, 1)
ys = (2, 5, 8, 10, 1, 2)
zs = (10, 3, 8, 9, 1, 8)

for zdir, x, y, z in zip(zdirs, xs, ys, zs):
    label = '(%d, %d, %d), dir=%s' % (x, y, z, zdir)
    ax.text(x, y, z, label, zdir)

# Demo 2: color
ax.text(9, 0, 0, "red", color='red')

# Demo 3: text2D
# Placement 0, 0 would be the bottom left, 1, 1 would be the top right.
ax.text2D(0.05, 0.95, "2D Text", transform=ax.transAxes)

# Tweaking display region and labels
ax.set_xlim(0, 10)W
ax.set_ylim(0, 10)
ax.set_zlim(0, 10)
ax.set_xlabel('X axis')
ax.set_ylabel('Y axis')
ax.set_zlabel('Z axis')

plt.show()
```

![text](https://matplotlib.org/_images/text3d_demo.png)
