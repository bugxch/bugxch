# 字节序


大端和小端，到底是什么意思？

<!--more-->

![](https://github.com/bugxch/blogpics/blob/master/201807/zijiexu.png?raw=true)

字节序是数据在内存中存放顺序。字节分高有效位和低有效位，最左边是高位最右边是低位。我们按照地址生长的方向依次往内存写入数据，如果高有效位的数据先写入就是 big endian，如果低有效位先写入就是 little endian。字节序仅仅与数据的存储方式有关，与语言没有关系，没有 java 字节序或者 C 字节序的区分。

举个例子，存入一个 4 字节的数据 0x0A0B0C0D，最高有效位的字节是 0A，最低有效位的字节是 0D，如果在内存存储中，0A 位于连续内存最小的那个就是 big endian，反之如果最小地址存储 0D 就是 little endian。

![big endian](https://upload.wikimedia.org/wikipedia/commons/thumb/5/54/Big-Endian.svg/320px-Big-Endian.svg.png)

big endian 存放顺序是最高有效位，次高有效位，一直顺序存储到最低有效位，数据和内存都是按照顺序依次连续存储的。little endian 也是依次存储，但是高低位刚好相反，如下图所示

![little endian](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ed/Little-Endian.svg/320px-Little-Endian.svg.png)

需要注意的是，无论是 4 个字节还是 8 个字节，存储的顺序都是在**连续内存**上**依照**高位到低位或者低位到高位的**顺序**读取和写入，比如我们有 3 个字节 0x123456，如果内存中保存的数据从小内存地址到大内存地址依次位 0x12,0x34, 0x56，那么就是 big endian，反之是 little endian。

附上 C 语言测试字节序的程序

```
void judge_endian(void)
{
    int a = 1; //存储0x00000001到内存

    if(*(&a) == 1){
	    //&a是a的地址，保存了1个字节的数据，如果为1，说明低有效位先保存，则为小端
        printf("Little Endian\n");
    } else {
        printf("Big Endian\n");
    }
}
```
