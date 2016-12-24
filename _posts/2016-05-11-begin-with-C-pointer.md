---
layout: post
title:  "从C指针开始"
date:   2016-05-11 14:39:55 +0800
categories: posts
---
书上云：

> 指针是一种保存地址的变量。

你也知道：
{% highlight C %}
int a = 1;
int *p = &a;            /* 将a的地址赋值给p */

*p = 3;                 /* 将p指向位置的数据修改为3 */
printf("%d\n", a);      /* 所以这里将输出3, 而不是1 */
{% endhighlight %}


1. **但是，地址究竟是什么？**
2. **既然都是地址，为什么指针还分类型呢？**

---

首先，你可能已经猜到，它是一个整数。
因为在计算机内部，所有数据都是以二进制表示的。

其次，为了简单，它是一个无符号整数。

那么，多大的整数合适呢？

`char`，`short`，`int`，`long`，`long long`？
太小肯定不行，越大越好吗？

实际上这个问题是“机器相关”的，比如`IA32`的最大的内存寻址是32位，所以32位整数就正好够用。
而`x86-64`最大寻址48位，所以用64位整数。

C语言数据类型`long`在32位机器下是4字节，64位机器下是8字节。所以通常`unsigned long`就是地址的类型了。

PS：这也是32位系统最多识别4GB内存的原因。

---

第二个问题，我们来看内存中数据的存储形式。

计算机上安装的内存可能不止一条，没条内存上又有很多内存颗粒，但是我们可以把内存抽象为一个字节数组。

但是数据的大小可能会大于一个字节，于是它被分散存储在连续的字节中。比如在x86机器上，`0x40490fda`会被存储为
`da 0f 49 40`。

    地址       值
    0x1000    da  <- p 指向这里
    0x1001    0f
    0x1002    49
    0x1003    40

如图所示，现在`p = 0x1000`，也就是起始地址。但是，`*p`究竟等于`0xda`还是`0x0fda`还是`0x40490fda`呢？这就是指针类型的作用了。

{% highlight C %}
*(unsigned int *)p == 0x40490fda
*(unsigned short *)p == 0x0fda
*(unsigned char *)p == 0xda
{% endhighlight %}

相对的，有一种特殊的指针`void *`不能被解引用

---

另外，对于这个例子

{% highlight C %}
*(float *)p == 3.141593
{% endhighlight %}

这一定不是巧合 Σ(ﾟДﾟ)

实际上，二进制数并不能单纯地理解为数字，而应该理解为一种模式（pattern）。为了运算方便，整数表示成补码的形式，而浮点数参见[IEEE 754](https://en.wikipedia.org/wiki/IEEE_754-1985)
有限的位数只能组成有限多个模式，所以计算机中的值都是离散的，这也是浮点误差产生的原因。

---

结论

> 指针保存了地址，地址就是一个无符号整数（位数通常等于系统位数）。
> 在编译之前，指针还携带了更多的信息，那就是它的类型。
> 指针类型包含了指针指向的数据块的大小（size），也就是字节数。

---
扩充

> 对指针进行加减运算，相当与对地址加减指针类型的size，这些转换都由编译器完成

最后，再贴一段代码，用来显示数据在内存中的形式。

{% highlight C %}
#include <stdio.h>

void show_bytes(const void *p, unsigned size)
{
	const unsigned char *cp = p;

	while (size--)
		printf(size == 0 ? "%0.2x\n" : "%0.2x ", *cp++);
}

int main()
{
	int a = 10000;
	float b = 3.141593;
	int *p = &a;
	float *q = &b;

	show_bytes(&a, sizeof(int));
	show_bytes(&b, sizeof(float));
	show_bytes(&p, sizeof(int *));
	show_bytes(&q, sizeof(float *));
	return 0;
}
{% endhighlight %}
