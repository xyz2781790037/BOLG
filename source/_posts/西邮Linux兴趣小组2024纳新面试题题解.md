---
title: 西邮Linux兴趣小组2024纳新面试题题解
comments: false
tags:
  - C语言
  - 纳新题
excerpt: 一个人可以被毁灭，但不能被打败。
toc: false
date: 2024-10-29 20:12:36
cover: 'https://images7.alphacoders.com/671/thumb-1920-671281.jpg'
---
> 本题目只作为西邮 Linux 兴趣小组 2024 纳新面试的有限参考。
> 为节省版面，本试题的程序源码省去了 #include 指令。
> 本试题中的程序源码仅用于考察 C 语言基础，不应当作为 C 语言「代码风格」的范例。
> 所有题目编译并运行于 x86_64 GNU/Linux 环境。
 

# 0. 聪明的吗喽​

>一个小猴子边上有 100 根香蕉，它要走过 50 米才能到家，每次它最多搬 50 根香蕉，（多了就拿不动了），它每走 1 米就要吃掉一根，请问它最多能把多少根香蕉搬到家里。

（提示：他可以把香蕉放下往返走，但是必须保证它每走一米都能有香蕉吃。也可以走到 n 米时，放下一些香蕉，拿着 n 根香蕉走回去重新搬 50 根。）

 **有一百根香蕉，因为他每次最多搬50根，所以将这100根香蕉分成a，b两组。所以把它分成前50根和后50根，在前50根的时候他每走一米，搬了a组，要吃一根，然后回去，又吃一根，然后将b组搬一米，吃一根，==所以每走一米要吃三根==，在前到16米时，它已经吃了48根，这时候要分第十七米了，它有两种选择:==1.直接拿着那个有50根香蕉的走，17米时吃了1根，剩了49根==；2。==继续像原来的方案一样，到十七米时也为*52 - 3 = 49根*==；之后就每走一米然后吃一根，直到50米走完，*49 - 33 = 16根*。所以最后答案为*16根*。**

# 1. 西邮Linux欢迎你啊​

请解释以下代码的运行结果。
```C
int main() {
 1. unsigned int a = 2024;

    for (; a >= 0; a--)
        printf("%d\n", printf("Hi guys! Join Linux - 2%d", printf("")));
    return 0;
 }
```
```C
Hi guys! Join Linux - 2024
......
```
 **此处为printf函数嵌套,用到了printf的返回值，printf返回值为打印元素的个数.并且进入嵌套时从嵌套外进去嵌套，读取时从最内侧的开始向外侧读取,由于最里面没有东西，所以返回值是0，最外面的那个看的是里面printf的字符个数，一共有24个字符，所以最后会打印2024.unsigned int 的取值范围为==0 - 2^32 - 1==.当a等于0时，它会从0直接到最大值，不会有负数，所以==for循环会无限执行下去==。**

# 2. 眼见不一定为实

输出为什么和想象中不太一样？

你了解 sizeof() 和 strlen() 吗？他们的区别是什么？

```C
int main() {
    char p0[] = "I love Linux";
    const char *p1 = "I love Linux\0Group";
    char p2[] = "I love Linux\0";
    printf("%d\n%d\n", strcmp(p0, p1), strcmp(p0, p2));
    printf("%d\n%d\n", sizeof(p0) == sizeof(p1), strlen(p0) == strlen(p1));
    return 0;
}
```

```bash
0
0
0
1
```

>**sizeof和strlen的区别：1.sizeof 是运算符，而strlen 是C语言库函数中的一个函数,且使用时包含==头文件<string.h>==；2.sizeof 操作符用于计算变量或类型的大小，一般单位为==字节==，通常用于计算内存大小。3.strlen是计算字符串长度的，遇到\0结束，==返回不包括\0==，即如果没有\0则会计算出随机值。**
 
**strcmp函数是C语言中的字符串比较函数，用于比较两个字符串的大小，且遇到\0或不同的字符会自动结束。==相等为0,第一个小于第二个，小于0==.这里的p1，p2，p0在\0处之前都相等，所以第一行需要打印的都为0；sizeof(第二条)会检索全部所以两个不相等，所以为0,strlen到\0结束，所以相等。**

# 3. 1.1 - 1.0 != 0.1

为什么会这样，除了下面给出的一种方法，还有什么方法可以避免这个问题？

```C
int main() {
    float a = 1.0, b = 1.1, ex = 0.1;
    printf("b - a == ex is %s\n", (b - a == ex) ? "true" : "false");
    int A = a * 10, B = b * 10, EX = ex * 10;
    printf("B - A == EX is %s\n", (B - A == EX) ? "true" : "false");
}
```
**float、double为浮点数，小数位数有限，比较容易损失精度。float类型数字在计算器中以二进制类型存储，==对于一些十进制小数，不能精确地用二进制表示，所以会导致1.1 - 1.0 != 0.1.==**

```C
    float a = 1.0, b = 1.1, ex = 0.1;
    printf("b - a == ex is %s\n", (b - a < ex + 0.0001) ? "true" : "false");
```
**此时会计算为正确。除此之外，还可以用Decimal,Decimal 不会把小数转换为二进制，而是就用十进制。**

# 4. 听说爱用位运算的人技术都不太差

解释函数的原理，并分析使用位运算求平均值的优缺点。

```C
int average(int nums[], int start, int end) {
    if (start == end)
        return nums[start];
    int mid = (start + end) / 2;
    int leftAvg = average(nums, start, mid);
    int rightAvg = average(nums, mid + 1, end);
    return (leftAvg & rightAvg) + ((leftAvg ^ rightAvg) >> 1);
}
```
   >leftAvg & rightAvg：获取 leftAvg 和 rightAvg 的共同部分。
    leftAvg ^ rightAvg：获取两个平均值不同的部分。
    ((leftAvg ^ rightAvg) >> 1)：将不同的部分右移一位，相当于除以2。
最后，将共同部分与右移后的不同部分相加，得到最终的平均值。
 
 **以下面的例子为例**
```C
int main()
{
    int arr[5] = {1, 2, 3, 4, 5};
    int a = average(arr, 0, 3);
    cout << a;
    return 0;
}
```
这个函数用递归的方式来求平均值，将数组分为左右两半，分别计算每一半的平均值。
`1 2 3 4 5`，第一次结束后为 `1 2 3` `4 5`,然后`1 2` `3`,4 5计算得 4（因为为int类型）,`1 2`计算为1,即 `1 3 4`
再分为`1 3` `4`,计算为2,4,最后算出为`3`。
>优点：高效，计算速度快。
>缺点：误差太大。
# 5. 全局还是局部!!!

先思考输出是什么，再动动小手运行下代码，看跟自己想得结果一样不一样 >-<

```C
int i = 1;
static int j = 15;
int func() {
    int i = 10;
    if (i > 5) i++;
    printf("i = %d, j = %d\n", i, j);
    return i % j;
}
int main() {
    int a = func();
    printf("a = %d\n", a);
    printf("i = %d, j = %d\n", i, j);
    return 0;
}
```
```bash
11 15 
11
1 15
```

**先要搞清楚定义：局部变量：==定义在函数体内部的变量，作用域仅限于函数体内部。离开函数体就会无效。再调用就是出错。==
全局变量：==所有的函数外部定义的变量，它的作用域是整个程序，也就是所有的源文件，包括.c和.h文件。==
在 C 或 C++等语言中，如果在全局变量和局部变量（如在 main 函数中定义的变量）有相同的名称，在局部作用域（即 main 函数内部）中会优先使用局部变量，局部变量会“压制”全局变量。这是因为编译器在查找变量时，会首先在当前的局部作用域中查找，如果找到了同名变量就使用它，而不会使用全局作用域中的同名变量。所以，在func函数里，i被重新定义，优先使用i = 10,i++后为11,a = 11 % 15 = 11；==在main函数里没有i，所以会用全局变量==，i = 1;**

# 6. 指针的修罗场：改还是不改，这是个问题

指出以下代码中存在的问题，并帮粗心的学长改正问题。

```C
int main(int argc, char **argv) {
    int a = 1, b = 2;
    const int *p = &a;
    int * const q = &b;
    *p = 3, q = &a;
    const int * const r = &a;
    *r = 4, r = &b;
    return 0;
}
```
**==第一个定义的是常量指针，它的值无法更改，但地址可以更改。
第二个定义的是指针常量，它的地址无法更改，但值可以更改。==**

```C
p = &b, *q = 4;
```
**==第三个为常量指针常量，它的值无法更改，且它的地址无法更改。所以不能改变地址和值。==**

# 7. 物极必反？

>你了解 argc 和 argv 吗，这个程序里的 argc 和 argv 是什么？
程序输出是什么？解释一下为什么。

```C
int main(int argc, char *argv[]) {
    while (argc++ > 0);
    int a = 1, b = argc, c = 0;
    if (--a || b++ && c--)
        for (int i = 0; argv[i] != NULL; i++)
            printf("argv[%d] = %s\n", i, argv[i]);
    printf("a = %d, b = %d, c = %d\n", a, b, c);
    return 0;
}
```
**==argv[]是命令行参数,argc是从命令行传给程序的参数个数(至少为1)==
在没有传入参数时 argc = 1,while(argc++ > 0),会让他从1加到最大，直到溢出为-2147483648，--a是先给a减一，a = 0;c-- ，先用c = 0,c = -1于是if不执行，所以执行最后。**

```C
a = 0, b = -2147483646, c = -1
```

# 8. 指针？数组？数组指针？指针数组？

在主函数中定义如下变量：

```C
int main() {
    int a[2] = {4, 8};
    int(*b)[2] = &a;
    int *c[2] = {a, a + 1};
    return 0;
}
```
**说说这些输出分别是什么？**

```C
a, a + 1, &a, &a + 1, *(a + 1), sizeof(a), sizeof(&a)
*b, *b + 1, b, b + 1, *(*b + 1), sizeof(*b), sizeof(b)
c, c + 1, &c, &c + 1, **(c + 1), sizeof(c), sizeof(&c)
```

```bash
0x7fffffffdb08 0x7fffffffdb0c 0x7fffffffdb08 0x7fffffffdb10 8 8 8
0x7fffffffdb08 0x7fffffffdb0c 0x7fffffffdb08 0x7fffffffdb10 8 8 8
0x7fffffffdb10 0x7fffffffdb18 0x7fffffffdb10 0x7fffffffdb20 8 16 8
```

>**由于题目可以看出来，第一个是数组，再看输出，a，a是一个数组，所以这里的a代表的是a数组的首地址，所以结果为a[0] (首地址)的地址；a + 1 是一个步长，int 类型步长是4个字节，所以为地址+4,为a[1]的地址；&a是给a取地址，所以为a整个的地址，但表示还是用a的首地址；&a  是指向整个数组的指针，当对这个指针执行  +1  操作时，它会根据整个数组的大小进行移动，所以移动8个字节；给a+1解引用，得到是a[1],即为8;sizeof是计算变量或类型的大小，一般单位为字节，此处为8字节；最后一个为指针，在64位系统中为8字节，32位为4字节。**

>**第二个是数组指针，** * b是取数组指针的第一个值，但这个值是&a,是a的地址；所以给* **b+1,结果是a的地址加一，即为加了四个字节；b 代表的是一个指向数组 a 的指针，所以是整个数组的地址；b + 1的话，就是给整个数组a的地址+1,它会根据整个数组的大小进行移动，所以移动8个字节；** *(*b + 1)就是给a + 1再解引用；sizeof(*b) **计算的是指针 b 指向的数组（即 a）的大小；sizeof(b) 返回的是指针 b 本身的大小，而b 是一个指向数组的指针，所以为8字节。**

>**最后一个是指针数组，所以输出的是指向 c 数组的地址，c 的首地址是指向a的地址；c + 1为c的第二个元素的地址，即指向a + 1地址的地址；&c 的类型为数组指针，即指向包含整个a的数组的指针；&c + 1为在指向a这个数组的指针再走一步此时为16字节；解引用(c + 1)将得到 c[1]，即 a + 1，这是指向 a 数组中第二个元素（值为 8）的指针，继续解引用,所以会得到a[1],就是8；sizeof的c是整个地址的字节，这里一个a的地址为4,但c指向的a的地址，所以一个指向地址的地址为8个字节，所以指针数组字节大小为16；sizeof(&c) 返回的是指向数组 c 的指针的大小，为8字节。**

# 9. 嘻嘻哈哈，好玩好玩

在宏的魔法下，数字与文字交织，猜猜结果是什么？

```C
#define SQUARE(x) x *x
#define MAX(a, b) (a > b) ? a : b;
#define PRINT(x) printf("嘻嘻，结果你猜对了吗，包%d滴\n", x);
#define CONCAT(a, b) a##b

int main() {
    int CONCAT(x, 1) = 5;
    int CONCAT(y, 2) = 3;
    int max = MAX(SQUARE(x1 + 1), SQUARE(y2))
    PRINT(max)
    return 0;
}
```

```bash
嘻嘻，结果你猜对了吗，包11滴
```
**这里CONCAT会使x1 = 5，y2 = 3;定义没有加括号（应该为这么定义#define SQUARE(x) ((x) * (x))），所以这里会写成5 + 1 x 5 + 1 = 11,令一个为9,max是判断谁更大，然后输出大的值即为11.**

# 10. 我写的排序最快

写一个 your_sort 函数，要求不能改动 main 函数里的代码，对 arr1 和 arr2 两个数组进行升序排序并剔除相同元素，最后将排序结果放入 result 结构体中。

```C
int main() {
    int arr1[] = {2, 3, 1, 3, 2, 4, 6, 7, 9, 2, 10};
    int arr2[] = {2, 1, 4, 3, 9, 6, 8};
    int len1 = sizeof(arr1) / sizeof(arr1[0]);
    int len2 = sizeof(arr2) / sizeof(arr2[0]);

    result result;
    your_sort(arr1, len1, arr2, len2, &result);
    for (int i = 0; i < result.len; i++) {
        printf("%d ", result.arr[i]);
    }
    free(result.arr);
    return 0;
}
```

```Cpp
typedef struct result
{
  int *arr;
  int len;
} result;//定义结构体
void your_sort(int arr1[], int len1, int arr2[], int len2, result *result)
{
  int str[19], x = 0;
  for (int i = 0; i < len1; i++)
  {
    str[x++] = arr1[i];
  }
  for (int i = 0; i < len2; i++)
  {
    str[x++] = arr2[i];
  }//将两个数组合并
  sort(str, str + len1 + len2);//排序
  result->arr = (int *)malloc((len1 + len2) * sizeof(int));//分配动态内存
  if (result->arr == NULL)
  {
    cout << "error!" << endl;
    exit(1);
  }
  int x1 = 0;
  for (int i = 0; i < x; i++)
  {
    if (i == 0 || str[i] != str[i - 1])//剔除相同元素
    {
      result->arr[x1++] = str[i];
    }
  }
  result->len = x1;
}
int main()
{
  int arr1[] = {2, 3, 1, 3, 2, 4, 6, 7, 9, 2, 10};
  int arr2[] = {2, 1, 4, 3, 9, 6, 8};
  int len1 = sizeof(arr1) / sizeof(arr1[0]);
  int len2 = sizeof(arr2) / sizeof(arr2[0]);

  result result;
  your_sort(arr1, len1, arr2, len2, &result);
  for (int i = 0; i < result.len; i++)
  {
    printf("%d ", result.arr[i]);
  }
  free(result.arr);//释放多余的内存
  return 0;
}
```

# 11. 猜猜我是谁

在指针的迷宫中，五个数字化身为神秘的符号，等待被逐一揭示。

```C
int main() {
    void *a[] = {(void *)1, (void *)2, (void *)3, (void *)4, (void *)5};
    printf("%d\n", *((char *)a + 1));
    printf("%d\n", *(int *)(char *)a + 1);
    printf("%d\n", *((int *)a + 2));
    printf("%lld\n", *((long long *)a + 3));
    printf("%d\n", *((short *)a + 4));
    return 0;
}
```

```bash
0
2
2
4
2
```
>****==void *a[] = {(void *)1, (void *)2, (void *)3, (void *)4, (void *)5}; 定义了一个void类型的数组 其中每个元素都是void类型的值 即每个元素都是一个十六进制的整形，且为小端储存==;
>第一个这里你获取数据，但`(char *)a + 1`是a的首地址强制转化为char类型，由于char只占一个字节，+1并取得第二个字节，这里指针为8字节，所以一个数字为0.5个字节，00000000000000`01` -> 000000000000`00`01,所以为0；
>==第二个表达式先将a转换为字符指针，然后再转换为整型指针，再解引用，此时为第一个数就是1==,1 + 1就是2；****
>后面三个与第一个同理，3.转化为int类型，+2刚好是八字节，即2；
>4.long long 为8字节，+3相当于走到数组第四个，即4；
>5.short2字节，+4为8字节，所以到第二个，即2；

# 12. 结构体变小写奇遇记

计算出 Node 结构体的大小，并解释以下代码的运行结果。

```C
union data {
    int a;
    double b;
    short c;
};
typedef struct node {
    long long a;//8字节
    union data b;//8字节
    void (*change)( struct node *n);//8字节
    char string[0];//柔性数组，内存后续会分配。
} Node;
void func(Node *node) {
    for (size_t i = 0; node->string[i] != '\0'; i++)
        node->string[i] = tolower(node->string[i]);
}
int main() {
    const char *s = "WELCOME TO XIYOULINUX_GROUP!";
    Node *P = (Node *)malloc(sizeof(Node) + (strlen(s) + 1) * sizeof(char));
    strcpy(P->string, s);//将s复制给P->string
    P->change = func;//将函数指针指向func
    P->change(P);//此时将大写全部转化为小写。
    printf("%s\n", P->string);
    return 0;
}
```
`Node *P = (Node *)malloc(sizeof(Node) + (strlen(s) + 1) * sizeof(char));`为柔性数组string分配内存空间。
结构体内存大小为`24`；

```C
welcome to xiyoulinux_group!
```
# 13. GNU/Linux (选做)

注：嘿！你或许对Linux命令不是很熟悉，甚至没听说过Linux。
但别担心，这是选做题，了解Linux是加分项，不了解也不扣分哦！

    你知道 ls 命令的用法与 / . ~ 这些符号的含义吗？
    你知道 Linux 中权限 rwx 的含义吗？
    请问你还懂得哪些与 GNU/Linux 相关的知识呢~
    
- ls 是一个用于列出目录内容的命令。常见用法包括：
> `ls`：列出当前目录的文件和子目录。
> ` ls -l`：以长格式显示文件和目录的详细信息，包括权限、拥有者、文件大小和修改时间。
> `ls -a`：显示所有文件，包括以.开头的隐藏文件。
- 
> /：根目录，所有文件和目录的起始点。
> .：当前目录的表示。
> ..：上一级目录的表示。
> ~：当前用户的主目录的快捷方式。

- 在 Linux 中，文件和目录的权限由三部分组成：用户、组和其他用户，每部分可以有三种权限：

> r（read）：读权限，允许查看文件内容。
> w（write）：写权限，允许修改文件内容。
> x（execute）：执行权限，允许执行文件（对目录来说，允许进入该目录）。
> eg:rwxr-xr--