# C语言补遗


前些天公司摸底 C 语言考试，得分比较难看，回来发了考试答案，这篇博客把我做错的题目拿出来理一理，补补课。

<!--more-->

- 判断对错：在定义数据结构时，没有特殊理由的话，都定义成四字节对齐；这样做可能浪费几个字节，但是不会出问题。

  > 这道题答案是对的，需要仔细 Google。

- 以下程序运行 (64 位系统) 后的输出结果是 6 5 8 5

  ```C
  int main()
  {
    char str1[] = "Hello";
    char str2[] = {'H', 'e', 'l', 'l', 'o'};
    char *p = str1;
    printf("%d %d %d %d\n", sizeof(str1), sizeof(str2), sizeof(p), strlen(p));
  }
  ```

  > 这道题我的答案是 6,5,4,5，原因在于 64 位的指针的长度大小是 8 个字节，而不是一般的 4 个字节。 考点为 字符串数组长度 (+1), 64 位指针长度 (8) 字符串数组 x 包含末尾的’\0’

- 如下程序, 在 64bit 系统运行输出为 24 4 8 16

  ```c
  struct s1
  {
      char a;
      int b;
      short c;
      double d;
  };
  int main(void)
  {
      printf("%d %d %d %d\n",
      sizeof(struct s1), offsetof(struct s1, b), offsetof(struct
      s1, c), offsetof(struct s1, d) ) ;
      //注: offsetof 为计算偏移量的宏
      return 0;
  }
  ```

  > 我的答案是 48,8,16,32，错误成了 2 倍。主要为结构体对齐规则, 这部分需要强化 一般考生都可以看到 b 按 4 字节对齐，需要填充 忽视 double 也需要 8 字节对齐 double 在 32 位 linux 平台下可能按照 4 字节对齐

- 如下指针计算, 结果为 4

  ```c
  int *p1 = (int *)0x500;
  int *p2 = (int *)0x510;
  printf(“%d\n”, (p2 - p1));
  ```

  > 我错成了 16，答案应该是 16/4，指针的减法，按照指针类型计算跨越的步长

- 如下程序, 请问输出多少？ -128,128

  ```c
  void main(void)
  {
  char x = 127;
  char a = x + 1;
  int b = x + 1;
  printf("%d %d", a, b);
  }
  ```

  > 我的答案是 128,128，整型加法隐式提升到 int 然后运算，所以 x+1 就是 128 溢出出现在赋值运算，第一个溢出了，第二个没有。没有考虑到第一个溢出之后的数据显示问题

- 如下程序片段, 输出为: 2 7

  ```c
  int *p;
  int x[3][3] = {1, 2, 3, 4, 5, 6, 7, 8, 9};
  p = x[0] + 1;
  printf("%d %d\n", *p, x[2][0]);
  ```

  > 我的答案是 4 7 。x[0] 是对二级指针解引用，类型是指向整型的指针，指向 1，+1 后，指向 x[0][1] 没争议

- 计算如下定义长度 (64 位系统)

  ```c
  typedef union {
      int a;
      long b;
      char c[6];
  }un;
  sizeof(un) = 8
  ```

  > 我的答案是 48，联合体的内存大小不会算，共用体的占用空间的大小，按照其最大成员算 本题中 long 是 8 字节

- 请问输出多少

  ```c
  union packet
  {
      struct packet_bit
      {
          unsigned char a:2;
          unsigned char b:3;
          unsigned char c:4;
      } bit;
      int i;
  } data;
  
  int main()
  {
      data.i = 0;
      data.bit.a = 1;
      data.bit.b = 2;
      data.bit.c = 0xF;
      printf("0x%04x\n", data.i);
  }
  ```

  > 我的答案是 0xf，这道题完全是蒙出来的，考察的是位域的概念，位域结构体，按地址从低到高依次存储 a、b、c 一个规则大家可能都不熟悉： 一个位域必须存储在同一个字节中，不能跨两个字节 所以 a 和 b 储存在一个字节，c 再进来存不下，所以 c 单独存放在一个字节 再就是注意输出格式

- 如下程序, 请写出打印结果 A b

  ```c
  void fun(char *a, char *b)
  {
      a = b;
      (*a)++;
  }
  void main()
  {
      char c1 = 'A', c2 = 'a';
      char *p1 = &c1;
      char *p2 = &c2;
      fun(p1, p2);
      printf("%c %c\n", c1, c2);
  }
  ```

  > 我的结果是 a b， 函数入参是按值传递的，只有通过传递地址才能改变函数外部的值，这里有一个陷阱，使用的地址，实际是从参数 b 传进来的，所以改变的也是参数 b 指向的值 。这道题不该错，因为在 fun 函数里，a 的值已经是 b 了，所以 a 指向的就是 b 指向的地址，因此改变的就是 c2 指向的字母啊

- 下面函数的输出是 token3 = 4

  ```c
  #define paster( n ) printf( "token" #n " = %d", token##n )
  void fun()
  {
      int token3 = 4;
      int tokenn = 3;
      paster( 3 );
  }
  ```

  > 我的答案是 token3 = 3，这是个知识盲点，可以参考 [C 语言宏定义 ## 连接符和 #符的使用](https://blog.csdn.net/dotphoenix/article/details/4345174)，# 将本身的字符替换之后再在两边加上双引号，## 是机械单纯得将两个 token 连接在一起

- 如下程序, 输出为 100008 100001 100004

  ```c
  struct test
  {
      int a;
      int b;
  };
  int main ()
  {
      struct test *p = (struct test *)0x100000;
      printf("%x %x %x\n", (p+1), (long)p+1, (int *)p +1);
  }
  ```

  > 我的答案是 0x100008,0x100001,0x11。这道题比较有意思，程序第 8 行已经说明了，p 的值就是 0x100000（也就是说 p 指向的地址是 0x100000），后面 (int *)p，说明了它指向的是 int 类型，那么 + 1 就是在原来的地址上一个 int 的字节数，也就是 0x100004。需要总结的是，指针前面的表明的是这个指针指向的数据类型。
