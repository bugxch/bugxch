# \$(cd \$(dirname $0);pwd)的解释


I love shell.

<!--more-->

![](https://github.com/bugxch/blogpics/blob/master/201807/i_love_bash.png?raw=true)

在很多shell脚本中，经常可以看到下面的语句

```shell
rootDir=$(cd $(dirname $0); pwd)
...
```

这个语句的作用是获取shell脚本所在目录的绝对路径，这个语句怎么理解？为什么不直接用`pwd`来获取当前路径呢？

### 语句解释

参考[explainshell.com - cd $(dirname $0); pwd](https://explainshell.com/explain?cmd=cd+%24%28dirname+%240%29%3B+pwd)的解释，拆解如下。

- `dirname`的功能是去掉文件路径名中的从右往左数的第一个`/`及其之后的所有文字，查看`dirname`的help信息可以看到如下的例子

  ```shell
  dirname /usr/bin/          -> "/usr"
  dirname dir1/str dir2/str  -> "dir1" followed by "dir2"
  dirname stdio.h            -> "."
  ```

- `$0`，这是bash shell脚本中的位置参数，用来表明输入到命令行中的命令本身。其余的还有`$1，$2`等等，分别表示输入到命令行中的命令后面带有的第一个参数和第二个参数，依次类推。比如下面的命令

  ```shell
  bash test9.sh 10 9
  ```

  其中的`$0`就是`test9.sh`，10和9分别是`$1`和`$2`。

- `pwd`，这个命令已经很熟悉了，就是打印当前的绝对路径。

-----

有了上面的分析，那**整个命令怎么理解呢**？举例说明，假如我们有如下的脚本`test.sh`在目录`~/DTS/code`下

``` shell
#/bin/bash
rootDir=$(cd $(dirname $0); pwd)
echo "rootDir $rootDir"
```

该脚本的功能就是寻找脚本所在目录下的所有的`.cc`文件。我们在命令行中运行该命令`./test.sh`，输出的结果是

```shell
bugxch@opensuse:~/DTS/code$ ./test.sh
rootDir /home/bugxch/DTS/code
```

请留意运行该脚本的时候的几个关键要素

- 调用脚本的路径。我们在目录`~/DTS/code/`下调用该脚本，也就是**当前目录**了。

- 调用脚本的命令。我们的命令是`./test.sh`.

结合上面的两条，此时`$(dirname $0)`的结果就是`.`，那么`cd $(dirname $0)`就是`cd .`，也就是切换命令到`~/DTS/code`，之后运行`pwd`，此时获得的就是脚本所在的绝对路径了。

### 为什么不用pwd？

请注意以下的基本事实

> 调用shell脚本，就是在调用脚本的**当前目录**下，**逐行执行**脚本中的**每一个命令**。

如果我们修改上面的脚本如下

```shell
#/bin/bash
rootDir=$(cd $(dirname $0); pwd)
echo "rootDir $rootDir"
echo "pwd $pwd"
```

并且在`~/DTS`下输入命令`code/test.sh`来运行这个脚本，输出结果如下

```shell
bugxch@opensuse:~/DTS$ code/test.sh
rootDir /home/bugxch/DTS/code
pwd /home/bugxch/DTS
```

第2个pwd不是脚本所在的目录，而是**我们输入命令的目录**。正如那个基本事实所示，在该目录下运行该脚本相当于逐行敲入脚本中每一句之后执行，所以在`~/DTS`调用脚本中的`pwd`，就相当于在该目录下敲入`pwd`，因此结果就是当前路径。

之所以不能直接使用`pwd`获取脚本所在目录，是因为如果在脚本目录之外调用该脚本，返回的是调用命令所在的目录而不是脚本所在目录。
