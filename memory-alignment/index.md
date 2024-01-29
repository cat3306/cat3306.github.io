# 内存对齐


# 32位与64位

**地址总线**

`CPU`要想读取程序中数据，需要地址总线，通过`MMU`将虚拟地址转换成物理的内存地址。如果地址总线有8根，那么`CPU`的最大寻址空间是`2^8` 地址[0,255]，则表示最大可用的内存为`256 byte`;同理如果想要寻址`4G`的内存，则需要地址总线32根。

**数据总线**

数据总线是内存数据流向`CPU`的通道，对应的如果想要一次性把地址中的数据一起传输，就需要和地址总线对齐如：32位地址总线:32位数据总线，64位地址总线:64位数据总线。32位的一次性能操作4字节，64位的能一次性操作8字节长度。这里的字节长度其实就是机器字长

{{% figure class="center" src="/img/system/1.png" alt="img" %}}

**总结**

所谓的32位操作系统和64位操作系统指的是数据总线能够一次性传输的位数，很显然64位的数据总线处理能力更高

# 内存布局

一块真实的物理内存条，有一面是几个黑色小方块组成。这个黑色的小方块就是`chip`，4G的内存条有4个`chip`，8G内存条有8个`chip`，而chip是由8个`bank`组成的三维结构

{{% figure class="center" src="/img/system/2.png" alt="img" %}}

一次操作选中连续的8个字节

{{% figure class="center" src="/img/system/3.png" alt="img" %}}

如果`address`不是8的倍数，就需要`cpu`取两次内存地址，再做拼接。例如:需要取1位后面8位的数据，则需要第一次从0开始取8个byte位，留下前面7个，第二次从8开始取8个byte位，留下前面一个，最终7+1得到想要的数据，但是必然有性能损耗。所以各个编程语言的编译器会把各式各样的数据类型放在合适的位置

{{% figure class="center" src="/img/system/4.png" alt="img" %}}

# 内存对齐

根据上面的内容可以知道内存对齐的目的就是要满足一次性将数据取出，避免不必要的性能浪费。内存对齐的条件如下

1. 类型的首地址是对齐边界的倍数
2. 类型的字节长度是对齐边界的倍数

{{% figure class="center" src="/img/system/5.png" alt="img" %}}

怎么确定每个数据类型的对齐边界呢？这和平台有关系，以`golang`为例，`PtrSize`指针宽度，`RegSize`寄存器宽度

| Arch Name | PtrSize | RegSize |
| :-------: | :-----: | :-----: |
|    386    |    4    |    4    |
|   amd64   |    8    |    8    |
|    arm    |    4    |    4    |
|   arm64   |    8    |    8    |

以上称为平台对应的最大对齐边界，数据类型的对齐边界是取类型大小与平台最大对齐边界中较小的一个。例如（:white_check_mark:表示最小的一个）

64位平台下

|  类型  |           大小           |         regsize         |
| :----: | :----------------------: | :---------------------: |
|  int8  | 1byte :white_check_mark: |          8byte          |
| int16  | 2byte:white_check_mark:  |          8byte          |
| int32  | 4byte:white_check_mark:  |          8byte          |
| int64  | 8byte:white_check_mark:  |          8byte          |
| string |          16byte          | 8byte:white_check_mark: |
| slice  |          24byte          | 8byte:white_check_mark: |

32位平台下

|  类型  |           大小           |         regsize         |
| :----: | :----------------------: | :---------------------: |
|  int8  | 1byte :white_check_mark: |          4byte          |
| int16  | 2byte:white_check_mark:  |          4byte          |
| int32  | 4byte:white_check_mark:  |          4byte          |
| int64  |          8byte           | 4byte:white_check_mark: |
| string |          8byte           | 4byte:white_check_mark: |
| slice  |          12byte          | 4byte:white_check_mark: |

# 结构的对齐边界

{{% figure class="center" src="/img/system/6.png" alt="img" %}}
