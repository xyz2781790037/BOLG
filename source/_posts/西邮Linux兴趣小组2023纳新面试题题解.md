---
title: 西邮Linux兴趣小组2023纳新面试题题解
comments: false
tags:
  - C语言
  - 纳新题
excerpt: 不论对任何困难都决不屈服！
toc: false
date: 2024-10-28 21:33:36
cover: 'https://images8.alphacoders.com/632/thumb-1920-632051.png'
---

学长寄语：长期以来，西邮Linux兴趣小组的面试题以难度之高名扬西邮校内。我们作为出题人也清楚的知道这份试题略有难度。请你动手敲一下代码。别担心，若有同学能完成一半的题目，就已经十分优秀。其次，相比于题目的答案，我们对你的思路和过程更感兴趣，或许你的答案略有瑕疵，但你正确的思路和对知识的理解足以为你赢得绝大多数的分数。最后，做题的过程也是学习和成长的过程，相信本试题对你更加熟悉的掌握C语言的一定有所帮助。祝你好运。我们东区逸夫楼FZ103见！
    

 - 本题目只作为西邮Linux兴趣小组2023纳新面试的有限参考。
 - 为节省版面，本试题的程序源码省去了#include指令。
 - 本试题中的程序源码仅用于考察C语言基础，不应当作为C语言「代码风格」的范例。
 - 所有题目编译并运行于x86_64 GNU/Linux环境。

# 0. 鼠鼠我啊，要被祸害了

>有1000瓶水，其中有一瓶有毒，小白鼠只要尝一点带毒的水，24小时
后就会准时死亡。至少要多少只小白鼠才能在24小时内鉴别出哪瓶水有毒？

**这个题考察二进制，用0,1表示死和生，也是喝与没喝，如果有一个小鼠，它可以表示为0,1,也就是2瓶水，那两只为4瓶水，我们可以给每瓶水编号，从0到999。用二进制表示，这需要10位（因为2^10=1024),即十只小鼠，每瓶水中位数是1,就让它喝，比如0001000101,让是1的小鼠喝水，24小时后看每只小鼠存活情况，就可以知道哪瓶水有毒了。**

# 1. 先预测一下~

按照函数要求输入自己的姓名试试~

```C
char *welcome() {
    // 请你返回自己的姓名
}
int main(void) {
    char *a = welcome();
    printf("Hi, 我相信 %s 可以面试成功!\n", a);
    return 0;
}
```

```C
char *welcome() {
    char *name = "hhh";
    return name;
}
int main(void) {
    char *a = welcome();
    printf("Hi, 我相信 %s 可以面试成功!\n", a);
    return 0;
}
```
***当你执行 char * name = "hhh"; 时，name 被赋值为指向字符串的首地址，即第一个字符 'h' 的地址，通过这个指针，你可以访问整个字符串。这里返回的是第一个h的地址，所以当后面使用 %s 来读取整个字符串。***

```C
char *welcome() {
    return 'hhh';
```

```C
char *welcome() {
    static char name[] = "hhh";
    return name;
   ```
   这是另外的两个方法。
# 2. 欢迎来到Linux兴趣小组

有趣的输出，为什么会这样子呢~
```C
int main(void) {
    char *ptr0 = "Welcome to Xiyou Linux!";
    char ptr1[] = "Welcome to Xiyou Linux!";
    if (*ptr0 == *ptr1) {
      printf("%d\n", printf("Hello, Linux Group - 2%d", printf("")));
    }
    int diff = ptr0 - ptr1;
    printf("Pointer Difference: %d\n", diff);
}
```

```bash
Hello, Linux Group - 2023
Pointer Difference: 1431667956
```
**if语句内指针解引用前一个指向常量字符串的‘w'。ptr1是数组，数组名即是首元素的地址，解引用ptr1是字符数组（字符串）的‘w',都是字符’w'，执行if下面的语句。此处为printf函数嵌套,用到了printf的返回值，printf返回值为打印元素的个数.并且进入嵌套时从嵌套外进去嵌套，读取时从最内侧的开始向外侧读取,由于最里面没有东西，所以返回值是0，最外面的那个看的是里面printf的字符个数，一共有23个字符，所以最后会打印2023。后面是两个地址相减，虽然两个解引用的东西一样，但地址不一样。所以值是随机的。**
# 3. 一切都翻倍了吗
   >请尝试解释一下程序的输出。
    请谈谈对sizeof()和strlen()的理解吧。
    什么是sprintf()，它的参数以及返回值又是什么呢？
```C
int main(void) {
    char arr[] = {'L', 'i', 'n', 'u', 'x', '\0', '!'}, str[20];
    short num = 520;
    int num2 = 1314;
    printf("%zu\t%zu\t%zu\n", sizeof(*&arr), sizeof(arr + 0),
           sizeof(num = num2 + 4));
    printf("%d\n", sprintf(str, "0x%x", num) == num);
    printf("%zu\t%zu\n", strlen(&str[0] + 1), strlen(arr + 0));
}
```
```bash
7       8       2
0
4       5
```

第一个是对arr数组先取地址，然后再进行解引用，得到的是数组的大小，这里有七个char类型的字符，所以这里大小是7;这里是arr + 0，就是arr的首地址，这里是指针
num是short类型所以是两个字节；后面用到了sprintf的相关知识，这里是将num转化为16进制然后再写入str这个数组，由于与num不相等，所以这里输出为0；后面+1,从第二个地址开始算，一共5个字符，所以为4,arr数组有\0前面有5个元素，所以为5。
>[sprintf详解](https://blog.csdn.net/Colorful___/article/details/132766101?ops_request_misc=%257B%2522request%255Fid%2522%253A%25229CD5AD7E-FF7B-4B49-9D85-397786DAFCC4%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=9CD5AD7E-FF7B-4B49-9D85-397786DAFCC4&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-132766101-null-null.142%5Ev100%5Epc_search_result_base1&utm_term=sprintf&spm=1018.2226.3001.4187)

# 4. 奇怪的输出

程序的输出结果是什么？解释一下为什么出现该结果吧~
```C
int main(void) {
    char a = 64 & 127;//01000000和01111111 -> 01000000 -> 64
    char b = 64 ^ 127;// 00111111 -> 63
    char c = -64 >> 6;//11000000 -> 11111111 -> -1;
    char ch = a + b - c;//计算答案为128,char类型范围为-128 ～ 127,-> -128
    printf("a = %d b = %d c = %d\n", a, b, c);
    printf("ch = %d\n", ch);
}
```
>[位运算详解](https://blog.csdn.net/MQ0522/article/details/129716834?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522D3D1213C-6F4E-4CD4-8CC6-971C8AAA9F9C%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=D3D1213C-6F4E-4CD4-8CC6-971C8AAA9F9C&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-129716834-null-null.142%5Ev100%5Epc_search_result_base1&utm_term=%E4%BD%8D%E8%BF%90%E7%AE%97%E7%AC%A6&spm=1018.2226.3001.4187)

```bash
a = 64 b = 63 c = -1
ch = -128
```

# 5. 乍一看就不想看的函数

>“人们常说互联网凛冬已至，要提高自己的竞争力，可我怎么卷都卷不过别人，只好用一些奇技淫巧让我的代码变得高深莫测。”

这个func()函数的功能是什么？是如何实现的？
```C
int func(int a, int b) {
    if (!a) return b;
    return func((a & b) << 1, a ^ b);
}
int main(void) {
    int a = 4, b = 9, c = -7;
    printf("%d\n", func(a, func(b, c)));//4 + 9 - 7 = 6;
}
```
```bash
6
```
**这个函数是计算两个值的加法运算，`&：计算进位`；`^：计算不带进位的和`。a & b取相同元素， << 1 相当于进位，a ^ b取不同元素，通过不断的递归，直到不再进位，最终求出加法最后的结果。**
# 6. 自定义过滤

请实现filter()函数：过滤满足条件的数组元素。

提示：使用函数指针作为函数参数并且你需要为新数组分配空间。
```C
typedef int (*Predicate)(int);
int *filter(int *array, int length, Predicate predicate,
            int *resultLength); /*补全函数*/
int isPositive(int num) { return num > 0; }
int main(void) {
    int array[] = {-3, -2, -1, 0, 1, 2, 3, 4, 5, 6};
    int length = sizeof(array) / sizeof(array[0]);
    int resultLength;
    int *filteredNumbers = filter(array, length, isPositive,
                                  &resultLength);
    for (int i = 0; i < resultLength; i++) {
      printf("%d ", filteredNumbers[i]);
    }
    printf("\n");
    free(filteredNumbers);
    return 0;
}
```

```C
typedef int (*Predicate)(int);
int *filter(int *array, int length, Predicate predicate, int *resultLength)
{
    int *filteredNumbers = (int *)malloc(length * sizeof(int));
    if (filteredNumbers == NULL)
    {
        cout << "error";
        exit(1);
    }
    int x = 0;
    for (int i = 0; i < length; i++)
    {
        if (predicate(array[i]))//判断数是不是大于0
        {
            filteredNumbers[x++] = array[i];
        }
    }
    *resultLength = x;
    return filteredNumbers;
}
int isPositive(int num) { return num > 0; }
```

# 7. 静…态…
   >如何理解关键字static？
    static与变量结合后有什么作用？
    static与函数结合后有什么作用？
    static与指针结合后有什么作用？
    static如何影响内存分配?

>**static是 C/C++中的关键字之一，是常见的函数与变量的修饰符，它常被用来控制变量的存储方式和作用范围。 `1.`如果静态变量没有显式初始化，编译器会自动将其初始化为零,用 static 声明的变量在程序的整个运行期间存在。`2.`无论变量是在函数内还是外部，静态变量在程序开始时分配内存，并在程序结束时释放。`3.`静态变量通常存储在静态区，这意味着它们不会随着函数调用的结束而被销毁。`4.`在函数内部声明的静态变量，其作用域仅限于该函数，但生命周期却是全程有效的,全局静态变量的作用域限制在定义它的文件中，其他文件无法访问。**
 - 修饰局部变量（称为静态局部变量）
 - 修饰全局变量（称为静态全局变量）
 - 修饰函数（称为静态函数）
> [static的详解](https://blog.csdn.net/weixin_45031801/article/details/134215425?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522B7C35F5F-27DA-4443-AD82-B4D3E06A8A0C%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=B7C35F5F-27DA-4443-AD82-B4D3E06A8A0C&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-134215425-null-null.142%5Ev100%5Epc_search_result_base1&utm_term=static&spm=1018.2226.3001.4187)
# 8. 救命！指针！
>数组指针是什么？指针数组是什么？函数指针呢？用自己的话说出来更好哦，下面数据类型的含义都是什么呢？
```C
int (*p)[10];
const int* p[10];
int (*f1(int))(int*, int);
```
***第一个是数组指针，是指向数组的指针，里面存放的是地址；
第二个是常量指针数组，是一个数组，其中每个元素都是指向变量的指针，且值无法更改。
第三个是函数指针，返回的函数将返回一个 int 类型的值，并接受一个 int* 和一个 int 类型的参数。***
# 9. 咋不循环了

程序直接运行，输出的内容是什么意思？
```C
int main(int argc, char* argv[]) {
    printf("[%d]\n", argc);
    while (argc) {
      ++argc;
    }
    int i = -1, j = argc, k = 1;
    i++ && j++ || k++;
    printf("i = %d, j = %d, k = %d\n", i, j, k);
    return EXIT_SUCCESS;//return EXIT_SUCCESS相当于return 0
}
```
==argv[]是命令行参数,argc是从命令行传给程序的参数个数(至少为1)==
在没有传入参数时 argc = 1,先给argc加，会让他从1加到最大，直到溢出为-2147483648,然后会继续执行，直到argc = 0，跳出while循环。i++,i = 0,j = argc = 0,j++,j = 1,k++,k = 2.

```bash
i = 0, j = 1, k = 2
```

# 10. 到底是不是TWO
```C
#define CAL(a) a * a * a
#define MAGIC_CAL(a, b) CAL(a) + CAL(b)
int main(void) {
  int nums = 1;
  if(16 / CAL(2) == 2) {
    printf("I'm TWO(ﾉ>ω<)ﾉ\n");
  } else {
    int nums = MAGIC_CAL(++nums, 2);
  }
  printf("%d\n", nums);
}
```
这里是宏定义没加括号，所以变成了
>16 / 2 * 2 * 2 = 32

所以这里不会执行I'm TWO(ﾉ>ω<)ﾉ，else里重新定义了一个新的nums，==出了else,生命周期就结束了==，所以不会影响结果，即nums = 1;

# 11. 克隆困境

试着运行一下程序，为什么会出现这样的结果？

>直接将s2赋值给s1会出现哪些问题，应该如何解决？请写出相应代码。
```C
struct Student {
    char *name;
    int age;
};

void initializeStudent(struct Student *student, const char *name,
                       int age) {
    student->name = (char *)malloc(strlen(name) + 1);
    strcpy(student->name, name);
    student->age = age;
}
int main(void) {
    struct Student s1, s2;
    initializeStudent(&s1, "Tom", 18);
    initializeStudent(&s2, "Jerry", 28);
    s1 = s2;
    printf("s1的姓名: %s 年龄: %d\n", s1.name, s1.age);
    printf("s2的姓名: %s 年龄: %d\n", s2.name, s2.age);
    free(s1.name);
    free(s2.name);
    return 0;
}
```

```C
Student s3 = s1;
  s1 = s2;
  s2 = s3;
```
**直接将s1 = s2，会使s1本来指向的内存空间，使其无法正常释放。**
# 12. 你好，我是内存

作为一名合格的C-Coder，一定对内存很敏感吧~来尝试理解这个程序吧！
```C
struct structure {
    int foo;//占四个字节
    union {
      int integer;
      char string[11];
      void *pointer;
    } node;//偏移为4，共16字节
    short bar;
    long long baz;
    int array[7];
};
int main(void) {
    int arr[] = {0x590ff23c, 0x2fbc5a4d, 0x636c6557, 0x20656d6f,
                 0x58206f74, 0x20545055, 0x6577202c, 0x6d6f636c,
                 0x6f742065, 0x79695820, 0x4c20756f, 0x78756e69,
                 0x6f724720, 0x5b207075, 0x33323032, 0x7825005d,
                 0x636c6557, 0x64fd6d1d};
    printf("%s\n", ((struct structure *)arr)->node.string);
}
```
union 的对齐数是由其成员中`最大的对齐`要求决定的。所以此时的union 的对齐数是8,中间还留了4个字节，从后开始转化，即`0x636c6557`，此时为小端序，所以是`57` `65` `6c` `63`-> W e l c直到`0x7825005d`,的`5d`  ,`00` 结束。

```bash
Welcome to XUPT , welcome to Xiyou Linux Group [2023]
```

# 13. GNU/Linux (选做)

>注：嘿！你或许对Linux命令不是很熟悉，甚至你没听说过Linux。但别担心，这是选做题，了解Linux是加分项，但不了解也不扣分哦！
你知道cd命令的用法与 / . ~ 这些符号的含义吗？

>1.切换到指定目录：`cd /路径`
>2.返回上一级目录：`cd  ..`
>3.返回上上一级目录 ：`cd ../..`
>4.切换到当前用户的主目录：`cd ~`
>5.切换到之前的目录：`cd -`
