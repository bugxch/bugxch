# 《CPlusPlus Primer Plus》第九章习题及答案

假期做题，记录下。

<!--more-->

此处是复习题的答案，编程练习题的答案见[bugxch/Solutions_C-PrimerPlus](https://github.com/bugxch/Solutions_C-PrimerPlus)。

### Answer #1

a) 形参在函数调用时候创建，在函数返回时销毁，自动存储变量，无链接性，所以是自动变量；

b) 文件共享的变量具有外部链接性，所以使用静态存储外部链接性的变量，比如在A文件中定义，在B中使用extern关键字引用；

c) 内部链接性，静态存储变量，可以使用static修饰符，或者使用未命名的命名空间；

d) 无链接性，但是是静态存储变量，在函数内部使用static修饰符定义

### Answer #2

有一下几点区别：

1. using声明仅仅导入特定的名称，但是using编译命令将导入一个名称空间的所有名称；
2. 假如名称空间和声明区域定义了相同的名称。using声明中如果该名称与局部名称发生冲突，编译器会发出警告，但是using编译指令导入的而名称会被局部版本隐藏。

### Answer #3

```c++
#inlcude <iostream>
int main ()
{
    using double x;
    std::cout << "Enter valud: ";
    while (!(cin >> x)) {
        std::cout << "Bad input! Please enter a number: ";
        std::cin.clear();
        while (std::cin.get() != '\n')
            continue;
    }
    std::cout << "Value = " << x << std::endl;
    return 0;
}
```

### Answer #4

```c++
#inlcude <iostream>
int main ()
{
    using std::cout;
    using std::cin;
    using double x;
    cout << "Enter valud: ";
    while (!(cin >> x)) {
        cout << "Bad input! Please enter a number: ";
        cin.clear();
        while (cin.get() != '\n')
            continue;
    }
    cout << "Value = " << x << endl;
    return 0;
}
```

### Answer #5

因为两个函数的形参和顺序一样，仅仅是返回值不同，因此无法使用函数重载。如果在不同的文件中使用，这两个函数的作用域不同，有两种方式实现，

1. 可以使用static关键字，

```c++
static double average(int a, intb)
```

2. 将函数的声明和定义放在未命名的名称空间中

### Answer #6

```shell
10
4
0
Other: 10, 1
another(): 10, -4
```

### Answer #7

```shell
1
4, 1, 2
2
2
4, 1, 2
2
```


