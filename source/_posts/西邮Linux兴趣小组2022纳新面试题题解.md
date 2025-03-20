---
title: 西邮Linux兴趣小组2022纳新面试题题解
comments: false
tags:
  - C语言
  - 纳新题
excerpt: 人生的价值，并不是用时间，而是用深度去衡量的。
toc: false
date: 2024-10-28 20:33:36
cover: 'https://images6.alphacoders.com/476/thumb-1920-476288.jpg'
---

>学长寄语：长期以来，西邮linux兴趣小组的面试题以难度之高名扬西邮校内。我们作为出题人也清楚的知道这份试题略有难度。请你动手敲一敲代码。别担心，若有同学能完成一半的题目，就已经十分优秀。其次，相比于题目的答案，我们对你的思路和过程更感兴趣，或许你的答案略有瑕疵，但你正确的思路和对知识的理解足以为你赢得绝大多数的分数。最后，做题的过程也是学习和成长的过程，相信本试题对你更加熟悉地掌握C语言一定有所帮助。祝你好运。我们东区逸夫楼FZ103见！

- 本题目只作为西邮Linux兴趣小组2022纳新面试的有限参考。
-  为节省版面，本试题的程序源码省去了#include指令。
-  本试题中的程序源码仅用于考察C语言基础，不应当作为C语言「代码风格」的范例。
 -  题目难度随机排列。
 -  所有题目编译并运行于x86_64 GNU/Linux环境。

# 0. 我的计算器坏了？！

  >2^10 = 1024 对应于十进制的4位，那么 2^10000 对应于十进制的多少位呢?

  **2^10 = 1024,就是lg 2^n = n * lg 2所以公式为`[nlg2] + 1`，n = 10000，大概为3011；**
# 1. printf还能这么玩？

   尝试着解释程序的输出。
```C
int main(void) {
  if ((3 + 2 < 2) > (3 + 2 > 2))
    printf("Welcome to Xiyou Linux Group\n");
  else
    printf("%d\n", printf("Xiyou Linux Group - 2%d", printf("")));
}
```
3 + 2 = 5 < 2为假，所以是0, 5 > 2为真，所以是1,0 < 1,所以执行else  
此处为printf函数嵌套,用到了printf的返回值，printf返回值为打印元素的个数.由于最里面没有东西，所以返回值是0，最外面的那个看的是里面printf的字符个数，一共有22个字符，所以最后会打印2022。

```bash
Xiyou Linux Group - 2022
```

# 2. 你好你好你好呀！

- 程序的输出有点奇怪，请尝试解释一下程序的输出吧。
- 请谈谈对`sizeof()`及`strlen()`的理解吧。

```C
int main(void) {
    char p0[] = "Hello,Linux";
    char *p1 = "Hello,Linux";
    char p2[11] = "Hello,Linux";
    printf("p0 == p1: %d, strcmp(p0, p2): %d\n", p0 == p1, strcmp(p0, p2));
    printf("sizeof(p0): %zu, sizeof(p1): %zu, sizeof(*p2): %zu\n",
           sizeof(p0), sizeof(p1), sizeof(*p2));
    printf("strlen(p0): %zu, strlen(p1): %zu\n", strlen(p0), strlen(p1));
}
```

```bash
p0 == p1: 0, strcmp(p0, p2): -1
sizeof(p0): 12, sizeof(p1): 8, sizeof(*p2): 1
strlen(p0): 11, strlen(p1): 11
```

***这里的p0是一个数组，所以此处p0代表的是首地址，p1也是地址，所以此处地址是不一样的，输出0.后面用strcmp函数，比较，两者一样，但p2的结尾没有以\0结束，所以p2大于p0，结果为0；
sizeof(p0)，是数组，大小为元素数，即12；p1为指针，所以大小为8；*p2在这里表示首地址指向的字符‘H’，所以为1；.strlen是计算字符串长度的，遇到\0结束，==返回不包括\0==，所以p0为11,p1也为11.***
## 3. 换个变量名不行吗？

请结合本题，分别谈谈你对C语言中「全局变量」和「局部变量」的「生命周期」理解。
```C
int a = 3;
void test() {
    int a = 1;
    a += 1;
    {
        int a = a + 1;
        printf("a = %d\n", a);
    }
    printf("a = %d\n", a);
}
int main(void) {
    test();
    printf("a= %d\n", a);
}
```

```bash
a = 32768
a = 2
a = 3
```
**因为在 int a = a + 1在作用域中， a 是未初始化的，==可能会导致随机值被打印==，出了作用域，a被销毁（生命结束）；a += 1,a = 2,函数外面打印2,main函数里没有定义a,所以用全局变量，即为3；**

# 4. 内存对不齐

`union`与`struct`各有什么特点呢，你了解他们的内存分配模式吗。

```C
typedef union {
    long l;
    int i[5];
    char c;
} UNION;
typedef struct {
    int like;//4字节
    UNION coin;//24字节，但要从8的倍数开始对齐，即第八个开始。
    double collect;//8字节，刚好从32开始对齐。
} STRUCT;
int main(void) {
    printf("sizeof (UNION) = %zu\n", sizeof(UNION)); 
    printf("sizeof (STRUCT) = %zu\n", sizeof(STRUCT));
}
```

```bash
sizeof (UNION) = 24
sizeof (STRUCT) = 40
```

> - **union 共用一段内存空间，内存大小就是最大成员的大小，但必须按照最大对齐数的整数倍来安排内存。long在这里为8,i为20,c 为1,最大为20,但要为8的倍数，所以是24；**
>  - **结构体大小为所有成员之和，且要有对齐数，所以为40;**
# 5. Bitwise

- 请使用纸笔推导出程序的输出结果。
- 请谈谈你对位运算的理解。

```C
int main(void) {
    unsigned char a = 4 | 7;//0100和0111 -> 0111
    a <<= 3;//0111000 -> 56
    unsigned char b = 5 & 7;//0101和0111 -> 0101
    b >>= 3;//0000000 -> 0
    unsigned char c = 6 ^ 7;//0110和0111 -> 0001
    c = ~c;//00000001 -> 11111110 -> -2 -> 254
    unsigned short d = (a ^ c) << 3;//11000110000 -> 1584 -> (char)48
    signed char e = -63;//11000001
    e <<= 2;//1100000100 -> 772 -> 4 -> 0x4
    printf("a: %d, b: %d, c: %d, d: %d\n", a, b, c, (char)d);
    printf("e: %#x\n", e);//%#x是一个格式说明符，用于printf函数来输出整数值的十六进制,并确保输出的十六进制数前面带有0x前缀。
}
```

```bash
a: 56, b: 0, c: 254, d: 48
e: 0x4
```
# 6. 英译汉

请说说下面数据类型的含义，谈谈`const`的作用。

 1. `char *const p`。
 2. `char const *p`。
 3. `const char *p`。

 1为指针常量，它的地址无法更改，但值可以更改。
 2,3为常量指针，指针可更改，但值不可更改。
>const的作用：
>1.修饰局部变量
>2.常量指针与指针常量
>3.修饰函数的参数
>4.修饰函数的返回值
>5.修饰全局变量

 >[const详解](http://blog.csdn.net/xingjiarong/article/details/47282255)


# 7. 汉译英

请用变量`p`给出下面的定义:

1. 含有10个指向`int`的指针的数组。
2. 指向含有10个`int`数组的指针。
3. 含有3个「指向函数的指针」的数组，被指向的函数有1个`int`参数并返回`int`。

`int* arr[10];`
`int (*a)[10];`
`int (*p[3])(int);`
**下来给个代码例子**

```C
// 被指向的函数
int add(int a, int b) {
    return a + b;
}
int multiply(int a, int b) {
    return a * b;
}
int subtract(int a, int b) {
    return a - b;
}
int main() {
    // 创建一个指向函数的指针数组// 定义函数类型
    int (*a[3])(int, int) = { add, multiply, subtract };
    return 0;
}
```

# 8. 混乱中建立秩序

你对排序算法了解多少呢?  
请谈谈你所了解的排序算法的思想、稳定性、时间复杂度、空间复杂度。

提示：动动你的小手敲出来更好哦~
>冒泡
>稳定性
>冒泡排序是稳定的排序算法。因为相等的元素不会交换位置。
>时间复杂度:
>最优时间复杂度：O(n)（当数列已经有序时）    
>最坏时间复杂度：O(n^2) 
>平均时间复杂度：O(n^2)
>空间复杂度:冒泡排序是原地排序算法，空间复杂度为 O(1)。

```C
void bubblesort(int arr[], int len)
{
    for (int i = 0; i < len - 1; i++)
    {
        for (int j = 0; j < len - i - 1; j++)
        {
            if (arr[j] > arr[j + 1])
            {
                swap(arr[j], arr[j + 1]);
            }
        }
    }
}
```
>选择
>稳定性:选择排序是不稳定的排序算法。因为相同元素在排序过程中可能会改变相对位置。
>时间复杂度:平均时间复杂度：O(n^2)
>空间复杂度:选择排序是原地排序算法，空间复杂度为 O(1)。

```C
void quicksort(int arr[], int len)
{
    for (int i = 0; i < len; i++)
    {
        for (int j = i + 1; j < len; j++)
        {
            if (arr[i] > arr[j])
            {
                swap(arr[i], arr[j]);
            }
        }
    }
}
```
>快速
>稳定性：不稳定的排序算法。因为相同元素在排序过程中可能会改变相对位置。
>时间复杂度
>最优时间复杂度：O(n log n)
>最坏时间复杂度：O(n^2)（当选择的基准元素是最大或最小值时）
>平均时间复杂度：O(n log n)
>空间复杂度
>最优空间复杂度：O(log n)（递归调用栈）
>最坏空间复杂度：O(n)（当每次划分极不平衡时)
>平均空间复杂度：O(log n)

```C
void quicksort(int arr[], int left, int right)
{
    if (left > right)
        return;
    int i = left, j = right;
    int temp = arr[i];
    while (i != j)
    {
        while (i < j && arr[j] >= temp)
        {
            j--;
        }
        while (i < j && arr[i] <= temp)
        {
            i++;
        }
        if (i < j)
        {
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[left], arr[i]);
    quicksort(arr, left, i - 1);
    quicksort(arr, i + 1, right);
}
```

# 9. 手脑并用

请实现ConvertAndMerge函数：  
拼接输入的两个字符串，并翻转拼接后得到的新字符串中所有字母的大小写。

提示:你需要为新字符串分配空间。

```C
char* convertAndMerge(/*补全签名*/);
int main(void) {
    char words[2][20] = {"Welcome to Xiyou ", "Linux Group 2022"};
    printf("%s\n", words[0]);
    printf("%s\n", words[1]);
    char *str = convertAndMerge(words);
    printf("str = %s\n", str);
    free(str);
}
```

**代码如下：**

```C
char* convertAndMerge(char words[2][20])
{
    int len;
    len = strlen(words[0]) + strlen(words[1]);
    char* arr = (char*)malloc(sizeof(char) * (len + 1));
    strcpy(arr,words[0]);
    strcat(arr,words[1]);
    for(int i = 0;i<len;i++)
    {
        if(islower(arr[i]))
        {
            arr[i] -= 32;//arr[i] = toupper(arr[i]);
        }
        else if(isupper(arr[i]))
        {
            arr[i] += 32;//arr[i] = tolower(arr[i]);
        }
    }return arr;
}
```

# 10. 给你我的指针，访问我的心声

程序的输出有点奇怪，请尝试解释一下程序的输出吧。

```C
int main(int argc, char **argv) {
    int arr[5][5];
    int a = 0;
    for (int i = 0; i < 5; i++) {
        int *temp = *(arr + i);
        for (; temp < arr[5]; temp++) *temp = a++;
    }
    for (int i = 0; i < 5; i++) {
        for (int j = 0; j < 5; j++) {
            printf("%d\t", arr[i][j]);
        }
    }
}
```
>这里是地址没有变，而是将a的值放在这块地址，而这块地址代表的就是二维数组的值。这里arr的首地址为`0x7fffffffdac0`,而arr[5]的地址为`0x7fffffffdb24`中间差值正好为100,即25个int类型的数字，*temp = a = 0，一直加到4,但for循环继续执行，后面会加25 - 5 = 20 次，a就加到了25,下次地址就是25 --- 29，这次地址差了15,继续到45,然后加10.加5,所以一直到75,但二维数组的值就为：
```bash
0       1       2       3       4       25      26      27      28      29      45      46      47      48      49      60      61    62       63      64      70      71      72      73      74
```

# 11. 奇怪的参数

你了解argc和argv吗？  
直接运行程序argc的值为什么是1？  
程序会出现死循环吗？

```C
#include <stdio.h>
int main(int argc, char **argv) {
    printf("argc = %d\n", argc);
    while (1) {
        argc++;
        if (argc < 0) {
            printf("%s\n", (char *)argv[0]);
            break;
        }
    }
}
```
**==argv[]是命令行参数,argc是从命令行传给程序的参数个数(至少为1)== 在没有传入参数时 argc = 1;
argc会一直增加直到溢出为负数，此时输出结果为程序的路径名。**

# 12. 奇怪的字符

程序的输出有点奇怪，请尝试解释一下程序的输出吧。

```C
int main(int argc, char **argv) {
    int data1[2][3] = {{0x636c6557, 0x20656d6f, 0x58206f74},
                       {0x756f7969, 0x6e694c20, 0x00000000}};
    int data2[] = {0x47207875, 0x70756f72, 0x32303220, 0x00000a32};
    char *a = (char *)data1;
    char *b = (char *)data2;
    char buf[1024];
    strcpy(buf, a);//把a放进buf
    strcat(buf, b);//把b和a连接起来
    printf("%s \n", buf);
}
```
个人基本为小端序，`char *a = (char *)data1`将这里转化为一个字节，即两个数字，*a获取data1的首地址并一个一个转化为字符形式，0x`63` `6c` `65` `57` -> `57` `65` `6c` `63` -> `W e l c`,最后形成字符串。

```bash
Welcome to Xiyou Linux Group 2022 
```

# 13. 小试宏刀

- 请谈谈你对`#define`的理解。
- 请尝试着解释程序的输出。

```C
#define SWAP(a, b, t) t = a; a = b; b = t
#define SQUARE(a) a *a
#define SWAPWHEN(a, b, t, cond) if (cond) SWAP(a, b, t)
int main() {
    int tmp;
    int x = 1;
    int y = 2;
    int z = 3;
    int w = 3;
    SWAP(x, y, tmp);
    printf("x = %d, y = %d, tmp = %d\n", x, y, tmp);
    if (x > y) SWAP(x, y, tmp);//2 > 1 成立
    printf("x = %d, y = %d, tmp = %d\n", x, y, tmp);
    SWAPWHEN(x, y, tmp, SQUARE(1 + 2 + z++ + ++w) == 100);//1 + 2 + 3 + 4 * 1 + 2 + 4 + 5 = 21;所以为，x,y,tmp,0;
    printf("x = %d, y = %d\n", x, y, tmp);//x = y =2;
    printf("z = %d, w = %d, tmp = %d\n", z, w, tmp);//z = 4 后z++,z = 5;
}
```

```C
if(0) t = a;//不执行此项
a = b;//x = y = 2;
b = t;//y = tmp = 2;
```

```bash
x = 2, y = 1, tmp = 1
x = 1, y = 2, tmp = 2
x = 2, y = 2
z = 5, w = 5, tmp = 2
```

# 14. GNU/Linux命令 (选做)

你知道以下命令的含义和用法吗：

注：
嘿！你或许对Linux命令不是很熟悉，甚至你没听说过Linux。  
但别担心，这是选做题，不会对你的面试产生很大的影响！  
了解Linux是加分项，但不了解也不扣分哦！

- `ls`
- `rm`
- `whoami`

请问你还了解哪些GNU/Linux的命令呢。

> 恭喜你做到这里！你的坚持战胜了绝大多数看到这份试题的同学。  
> 或许你自己对答题的表现不满意,但别担心，请自信一点呐。  
> 坚持到达这里已经证明了你的优秀。  

-` ls` 是一个用于列出目录内容的命令。常见用法包括：
   >`ls`：列出当前目录的文件和子目录。
   >` ls -l`：以长格式显示文件和目录的详细信息，包括权限、拥有者、文件大小和修改时间。
   >`ls -a`：显示所有文件，包括以.开头的隐藏文件。

- `rm` 是一个用于删除文件和目录的命令。
   >`-f`：强制删除，忽略不存在的文件，不提示确认。
   >`-i`：交互模式，在删除每个文件之前提示确认。
   >`-r` ：递归删除，用于删除目录及其内容。
   >`-v`：显示详细输出，列出正在删除的文件。

- `whoami` 用于显示当前用户的用户名。