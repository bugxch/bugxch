# python排序


python基本操作知识积累。

<!--more-->

![python总结](https://geekswipe.net/wp-content/uploads/2014/10/Geekswipe-Python-Twitter-bot-Source-Photo-by-Karthikeyan-KC.jpg)

排序是一个基本的数据操作，本文列举下 python 和 numpy 中的基本排序操作，以备后查。

## 列表排序

列表排序常用`reverse/sort/sorted`这三种方法，举例如下

```python
In [28]: ls = [-1, 2, -4, 8, 3]
In [29]: ls.sort()
In [30]: ls
Out[30]: [-4, -1, 2, 3, 8]
In [31]: ls.sort(reverse=1)
In [32]: ls
Out[32]: [8, 3, 2, -1, -4]
In [33]: b = ls.sort()
In [34]: b
In [35]: print b
None #并不能生成新列表
```

使用`sort`是按照递增序列排序，使用参数`reverse`是递减序列排序，但是这样的方法改变了列表原来的值。

> **sort 与 sorted 区别**：
> sort 是应用在 list 上的方法，sorted 可以对所有可迭代的对象进行排序操作。list 的 sort 方法返回的是对已经存在的列表进行操作，而内建函数 sorted 方法返回的是一个新的 list，而不是在原来的基础上进行的操作。

如果不想改变原来的值，可以使用`sorted`方法，举例如下

```python
In [34]: b = sorted(ls)
In [35]: b
Out[35]: [-4, -1, 2, 3, 8] #生成新列表
```

## 字典排序

可以使用 sorted 对字典排序，`sorted`的函数原型如下所示

`sorted(iterable, cmp=None, key=None, reverse=False) --> new sorted list`，其中`iterable`代指可以迭代的数据类型，`key`表示按照什么排序，可以使用`lambda`函数，举例如下

```python
In [20]: x
Out[20]: {'A': 34, 'B': 23, 'C': 10, 'Y': 2}
In [21]: x.items()
Out[21]: [('A', 34), ('Y', 2), ('C', 10), ('B', 23)]
In [22]: y = sorted(x.items(),key = lambda item: item[1]) #按照元组的第2个元素排序

In [23]: y
Out[23]: [('Y', 2), ('C', 10), ('B', 23), ('A', 34)]
In [24]: y = sorted(x.items(),key = lambda item: item[0]) #按照元组的第1个元素排序
In [25]: y
Out[25]: [('A', 34), ('B', 23), ('C', 10), ('Y', 2)]
```

还有一种更简单的方法，使用 python 的`operator`模块，举例如下

```python
In [33]: from operator import itemgetter
In [34]: y = sorted(x.items(),key = itemgetter(0)) #取元组第一个元素
In [35]: y
Out[35]: [('A', 34), ('B', 23), ('C', 10), ('Y', 2)]
In [36]: y = sorted(x.items(),key = itemgetter(1)) #取元组第二个元素
In [37]: y
Out[37]: [('Y', 2), ('C', 10), ('B', 23), ('A', 34)]
```

字典排序的本质就是将字典的 items 对转换成列表，使用列表的排序方式来排序，`sorted`还有更复杂的功能，对于复杂的元组列表，可以根据元组中的任意一个元素排序，具体如下所示

```python
>>> student_tuples = [
...     ('john', 'A', 15),
...     ('jane', 'B', 12),
...     ('dave', 'B', 10),
... ]
>>> sorted(student_tuples, key=lambda student: student[2])   # sort by age
[('dave', 'B', 10), ('jane', 'B', 12), ('john', 'A', 15)]
```

## numpy 中的向量和矩阵的排序

numpy 中的排序方法与之前的类似，使用`sort`即可，不过 numpy 多了一个`axis`的选项，表示按行（axis = 1）还是按照列进行排序 (axis = 0)，具体举例如下

```pyhton
In [7]: a = np.array([[1,4], [3,1]])
In [8]: a.sort(axis =1)
In [9]: a
Out[9]: array([[1, 4],
       [1, 3]])
In [10]: a.sort(axis =0)
In [11]: a
Out[11]: array([[1, 3],
       [1, 4]])
```

如果需要获得排序的 index，可以使用`argsort`方法，比如

```python
In [16]: arr = np.array([1,3,2,6,3,4,-1])
In [17]: index = arr.argsort()
In [18]: index
Out[18]: array([6, 0, 2, 1, 4, 5, 3])
```

------

## 参考资料

- [Python dict sort 排序 按照 key，value - All About Python - SegmentFault](https://segmentfault.com/a/1190000004959880)
- [Sorting HOW TO — Python 3.6.3 documentation](https://docs.python.org/3/howto/sorting.html)
