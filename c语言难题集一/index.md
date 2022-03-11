# C语言难题集(一)


{{< admonition question 导语>}}

对于数组越界，你可想过其危险性的原因？

{{< /admonition >}}

```c
#include <stdio.h>

int main()
{
    int i = 0;
    int arr[10] = {0};
    for ( i = 0; i <= 12; i++)
    {
        arr[i] = 0;
        //请问在VS2019编译器下打印多少次hello？
        printf("hello\n");
    }
    return 0;
}
```

{{< admonition success 答案>}}

死循环打印hello。

这是由于 C 语言的栈机制导致的：

![image-20220311164645123](https://s2.loli.net/2022/03/11/Ybyl9A3Qf2nqevJ.png)

因此，当数组越界时将有可能访问到其他变量，在这里恰好是循环控制条件 i，这也就导致了死循环的发生！

值得一提的是，VS预留两个空间，GCC是连续空间，VC6.0预留一个空间。

{{< /admonition>}}

类似的还有下面这个例子：

```c
#include <stdio.h>

int main()
{
    char s[] = {"hello world"};
    char s1[10];
    printf("%s \n", s);
    printf("%s\n", s1);
    //请问在VS2019编译器下打印什么？
    return 0;
}
/*
输出：
hello world
烫烫烫烫烫烫烫烫烫烫hello world
*/
```

同样的道理，在打印s1时，由于未赋值，而 VS 会给其赋`0xcccc cccc`，也就是`烫`，又因为没有`\0`作为结尾，因此其会一直按栈区顺序寻找，直到遇到`\0`为止。

