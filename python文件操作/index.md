# python文件操作


在日常工作中，需要非常频繁地与数据打交道，将数据保存下来做数据比对，因此需要总结下python下的文本和数据的读写操作。

<!--more-->

# python文件读写

## 基础知识

在python中使用下面的语句打开一个文本

```python
f = open('workfile', 'w')
```

创建了一个文件对象`f`。其中的

- **读写方式**。`w`表示文本`workfile`的创建方式为写入，当然还有其他的读写方式，可以参考如下的表格

| "r"  | **read:** Open file for input operations. The file must exist. |
| ---- | ------------------------------------------------------------ |
| "w"  | **write:** Create an empty file for output operations. If a file with the same name already exists, its contents are discarded and the file is treated as a new empty file. |
| "a"  | **append:** Open file for output at the end of a file. Output operations always write data at the end of the file, expanding it. Repositioning operations ([fseek](http://www.cplusplus.com/fseek), [fsetpos](http://www.cplusplus.com/fsetpos), [rewind](http://www.cplusplus.com/rewind)) are ignored. The file is created if it does not exist. |
| "r+" | **read/update:** Open a file for update (both for input and output). The file must exist. |
| "w+" | **write/update:** Create an empty file and open it for update (both for input and output). If a file with the same name already exists its contents are discarded and the file is treated as a new empty file. |
| "a+" | **append/update:** Open a file for update (both for input and output) with all output operations writing data at the end of the file. Repositioning operations ([fseek](http://www.cplusplus.com/fseek), [fsetpos](http://www.cplusplus.com/fsetpos), [rewind](http://www.cplusplus.com/rewind)) affects the next input operations, but output operations move the position back to the end of file. The file is created if it does not exist. |

- **编码方式**。一般而言，默认读写文件使用的是字符串，字符串的编码方式视平台而定，也可以通过参数`b`来指定读写的数据为二进制文件，该模式仅仅用于没有字符的数据。

上面的基本语句有一个更优雅的语句方式

```python
with open('workfile') as f:
	read_data = f.read()
```

这样写的好处是文件可以自动关闭，即便有异常抛出也可以。

## python文件读写

这一段介绍python本身的文件读写功能。

### 文本文件

以之前的基础知识为例，读取文本文件很简单，使用文件对象的方法有

- `read`，直接读取文本的所有内容，也可以使用`f.read(size)`读取size字节的数据

```python
with open('workfile') as f:
	read_data = f.read()
    print(read_data)
```

- `readline`，逐行读取文件中每一行

```python
with open('workfile') as f:
	first_line = f.readline()
    sec_line = f.readline()
```

上面的语句依次读取文件的每一行。

- 遍历文本的每一行

```python
with open('files\stringdata.txt','r') as f:    
    for line in f:
        print(line)
```

这个会依次读取文件的每一行

- 将文本中的每一行读入一个列表中，列表的每个元素是文本的一行

```python
with open('files\stringdata.txt','r') as f:    
    s = f.readlines()
    print(s)
```

- 写入字符文本，仅仅需要一个函数`write`

```python
with open('files\stringdata.txt','w') as f:    
    f.write("hello world\n")
    f.write(str(10.255616))
```

文本内容是

```shell
hello world
10.255616
```

### 二进制文件

二进制文件的读写和文本文件读写的方法类似，最大的区别是打开的选项必须要加一个`b`。

- 写入二进制文件

```python
with open(filepath, 'wb') as f:
        f.write(output)
```

这个脚本将output的数字保存到filepath对应的文件里面。

- 读取二进制文件

```python
with open(filepath, 'rb') as f:
        content = f.read()
```

下面是读写二进制文件的一个例子

```python
import numpy as np

def save_bin(output,filepath):
    with open(filepath, 'wb') as f:
        f.write(output)
    return

def load_bin(filepath, dtype=np.float32):
    with open(filepath, 'rb') as f:
        content = f.read()
    data = np.frombuffer(content,dtype=dtype)
    return data

data_out = np.random.randn(1,10)
data_out = np.float32(data_out)

print(data_out)

save_bin(data_out, './datawrite.bin')

data_in = load_bin('./datawrite.bin', np.float32)

print(data_in)
```

其中使用numpy模块生成正态分布的10个数字，并且以float32的格式保存在文件里面，之后再读取出来。需要注意的是，**读取和写入文件的数据格式必须保持一致**，比如float16写入就float16数据格式读出，否则读出来的数据是错误的。我在ipython环境下随机运行的结果如下

```shell
In [44]: runfile('C:/Users/h00437182/Desktop/python/readfile.py', wdir='C:/Users/h00437182/Desktop/python')
[[ 0.5902766   0.19589928  0.12194731 -0.17657255 -0.4276924  -0.3527161
  -1.3733053  -0.7128385  -1.1449672   1.4563532 ]]
[ 0.5902766   0.19589928  0.12194731 -0.17657255 -0.4276924  -0.3527161
 -1.3733053  -0.7128385  -1.1449672   1.4563532 ]
```

## numpy文件读写

numpy是python的一个数据处理的模块，它的文件读写功能使用起来更加强大和便捷。在日常工作中，常常需要比对数据，因此有必要将数据保存成txt文档进行比对，此处的文件读写都是围绕数据展开。将数据保存在文本文件中是日常工作的一个大痛点和需求，下面的命令就可以实现这样的目标。

> numpy的数据读写是成对使用的。

- `savetxt`和`loadtxt`

  这两个命令只适用于一维或者二维数组，常见的使用格式如下

  ```python
  np.loadtxt(filename, dtype=int, delimiter=' ')
  np.savetxt(filename, data, fmt="%d",delimiter=' ')
  ```

  比如，下面的脚本，我们复用之前的脚本生成10个随机数，之后以小数点后4位的精度保留，之后再读出来

  ```python
  import numpy as np
  
  data_out = np.random.randn(1,10)
  print(data_out)
  
  np.savetxt('data.txt',data_out, fmt="%.4f", delimiter=' ')
  
  dataout = np.loadtxt('data.txt', delimiter=' ')
  print(dataout)
  ```

  文件`data.txt`中的数据是

  ```shell
  1.9229 -1.0319 0.0624 0.6372 -0.4520 0.7549 -1.6933 -0.2295 1.4575 -0.7399
  ```

- `tofile`和`fromfile`

  这两个命令的说明如下

  ```python
  ndarray.tofile(fid, sep="", format="%s")
  numpy.fromfile(file, dtype=float, count=-1, sep='')
  ```
  需要注意的是，使用fromfile读出的文件的数据是一维的，需要知道数据的维度才能恢复数据本来的样子，参考如下的用例

  ```python
  import numpy as np
  
  data_out = np.random.randn(1,10)
  data_out = np.float64(data_out)
  print(data_out)
  
  data_out.tofile('data.bin')
  dataout = np.fromfile('data.bin', dtype=np.float64)
  print(dataout)
  ```

- `save`和`load`

  这一对命令可以保存原始的数据，就是原封不动保存之前的narray的信息，使用load之后得到的数据就是原来的数据，参考如下的例子

  ```python
  import numpy as np
  
  data_out = np.random.randn(2,10)
  print(data_out)
  
  np.save('data.npy', data_out)
  dataout = np.load('data.npy')
  print(dataout)
  ```

  这个脚本的结果如下所示

  ```shell
  runfile('C:/Users/h00437182/Desktop/python/readfile.py', wdir='C:/Users/h00437182/Desktop/python')
  [[ 0.768138    1.09159732  1.45133552 -1.796178    1.19846175 -0.40864606
     0.73694781 -0.87850709 -2.29957767 -1.2550539 ]
   [-0.38308247  0.08059661 -1.84944623 -0.43883914  1.88381452  1.0455034
    -0.01678215  0.95006648  0.06723308 -0.6050553 ]]
  [[ 0.768138    1.09159732  1.45133552 -1.796178    1.19846175 -0.40864606
     0.73694781 -0.87850709 -2.29957767 -1.2550539 ]
   [-0.38308247  0.08059661 -1.84944623 -0.43883914  1.88381452  1.0455034
    -0.01678215  0.95006648  0.06723308 -0.6050553 ]]
  ```

  完美保存数据的精度和维度信息。

## 参考资料

- [7. Input and Output — Python 3.7.1 documentation](https://docs.python.org/3/tutorial/inputoutput.html)，python官方介绍输入输出的文档
- [Reading and Writing Files in Python (article) - DataCamp](https://www.datacamp.com/community/tutorials/reading-writing-files-python)，datamap的二进制文本读取
- [python - How to save and load numpy.array() data properly? - Stack Overflow](https://stackoverflow.com/questions/28439701/how-to-save-and-load-numpy-array-data-properly)

