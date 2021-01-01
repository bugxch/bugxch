# C语言中的宏


C 中有些特殊的宏定义，面试时候被问到，写个短文总结下。
<!--more-->

## 宏定义中的 #和## 连字符

这两个字符在宏定义中代表连接和替换，

- `#`紧跟字母表示对应字符的字符串化，将对应的字符转换成对应的字符串，比如`#hello`就是`"hello"`
- `##`表示将宏定义中的两个标识符连接在一起，组成一个新的标识符，类似胶水。它首先查看`##`两边的字符，是否有宏定义可以替换的字符串，替换之，之后将两个连接在一起。

下面举例说明。

```c
#include <stdio.h>
#define trace(x, format) printf(#x " = %" #format "\n", x)
#define trace2(i) trace(x##i, d)

int main(int argc, _TCHAR* argv[])
{
    int i = 1;
    char *s = "three";
    float x = 2.0;

    trace(i, d);                // 相当于 printf("x = %d\n", x)
    trace(x, f);                // 相当于 printf("x = %f\n", x)
    trace(s, s);                // 相当于 printf("x = %s\n", x)

    int x1 = 1, x2 = 2, x3 = 3;
    trace2(1);                  // 相当于 trace(x1, d)
    trace2(2);                  // 相当于 trace(x2, d)
    trace2(3);                  // 相当于 trace(x3, d)

    return 0;
}
```

又比如

```c
#define STACK_ADD_TASK_IRAM_POOL1(TASK_NAME, task_name, task_type){\
task_info_g[INDEX_##TASK_NAME].task_entry_func = \
    (osa_task_func_ptr) stack_##task_type##_task;\
}
```

上面的语句中，我们会将其中的`TASK_NAME`和`task_type`替换掉，并与前后的标识符相连接生成新的标识符。

## 宏定义中的 do{} while(0)

经常在宏定义中会看到如下的语句

```c
#define STACK_ADD_MULTI_TASK(multi_sys_max, TASK_NAME, task_name, task_type)  \
    do{                                                                       \
       if(multi_sys_max >= 1)                                                 \
       {                                                                      \
           STACK_ADD_TASK_1_CARD(TASK_NAME, task_name, task_type);            \
       }                                                                      \
    }while(0)
```

这个语句的特点是宏定义之后紧跟一个`do{...}while(0)`的结构，看起来颇为繁琐，**那这样的定义有什么好处呢？**

首先，C 中的宏定义，在预编译阶段就会将宏定义的结构替换掉，使用宏定义定义函数。在代码替换中肯定希望像使用定义的函数使用宏定义，比如上面的语句在代码中肯定是下面这样的

```c
STACK_ADD_MULTI_TASK(multi_sys_max, TASK_NAME, task_name, task_type);
```

所以注意到**宏定义的while(0)后面没有分号**。

然后，这个结构和`if..else...`的控制结构可以完美结合，对照上面的宏定义，一般能想到的宏定义的结果修改如下

```c
#define STACK_ADD_MULTI_TASK(multi_sys_max, TASK_NAME, task_name, task_type)  \
    {                                                                         \
       if(multi_sys_max >= 1)                                                 \
       {                                                                      \
           STACK_ADD_TASK_1_CARD(TASK_NAME, task_name, task_type);            \
       }                                                                      \
    }
```

如果正常替换之前的语句，替换之后的结果就是

```c
{                                                                         \
   if(multi_sys_max >= 1)                                                 \
   {                                                                      \
       STACK_ADD_TASK_1_CARD(TASK_NAME, task_name, task_type);            \
   }                                                                      \
};
```

注意最后的分号，**这一行编译不通过**，但是很明显如果换成`do{}while(0)`结构就不存在这个问题。

其次，如果使用宏定义多行语句，那么使用大括号的宏定义嵌套在`if...else...`的结构中会遇到问题，比如定义

```c
#define BAR(X) f(x); g(x)
```

如果用在下面的程序中就会出现语法错误

```c
if(true)
    BAR(x)
else
   {
       //do nothing
   }
```

但是使用`do{...}while(0)`结构就不会有这样的问题。

## 参考资料

- [C/C++ 宏定义中 #、##、#@的区别 - hellokandy 的博客 - CSDN 博客](http://blog.csdn.net/hellokandy/article/details/50592971)
- [c - Why use do {} while (0) in macro definition? - Stack Overflow](https://stackoverflow.com/questions/9495962/why-use-do-while-0-in-macro-definition)
- [c++ - Do you consider this technique “BAD”? - Stack Overflow](https://stackoverflow.com/questions/243967/do-you-consider-this-technique-bad)
- [c++ - Why use apparently meaningless do-while and if-else statements in macros? - Stack Overflow](https://stackoverflow.com/questions/154136/why-use-apparently-meaningless-do-while-and-if-else-statements-in-macros)
