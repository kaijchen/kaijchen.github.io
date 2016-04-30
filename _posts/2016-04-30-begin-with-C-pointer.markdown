---
layout: post
title:  "从C指针开始"
date:   2016-04-30 16:39:55 +0800
categories: Coding
---
书上云：

> 指针是一种保存地址的变量。

你也知道：
{% highlight C %}
int a = 1;
int *p;

p = &a;             /* 将a的地址赋值给p */
*p = 3;             /* 将p指向位置的数据修改为3 */
printf("%d\n", a);  /* 所以这里将输出3, 而不是1 */
{% endhighlight %}


1. **但是，地址究竟是什么？**
2. **既然都是地址，为什么指针还分类型呢？**

先来举个例子吧

{% highlight C %}
unsigned int a = 0x40490fda;    /* 十六进制无符号整数 40490fa */

unsigned int *p = &a;
unsigned short *sp = p;         /* 这边会出Warning，先忽略 */
unsigned char *cp = p;

printf("%x\n", *p);             /* %x十六进制 */
printf("%x\n", *sp);
printf("%x\n", *cp);
{% endhighlight %}

这段代码的运行结果是

    40490fda
    fda
    da

第二行`fda = 0fda`

找下规律，嗯，好像说明了什么。

接着上文，再来两行。

{% highlight C %}
float *fp = (float *)ip;
printf("%f\n", *fp);
{% endhighlight %}

结果

    3.141593

这一定不是巧合 Σ(ﾟДﾟ)

这时候，输出它们的地址（`%p`输出指针）

{% highlight C %}
printf("%p\n%p\n%p\n%p\n", p, sp, cp, fp);
{% endhighlight %}

结果可能会变化，但是4个值之间永远是相等的。
（我是64位系统，所以是64位无符号整数）。

    0x7ffe3b09377c
    0x7ffe3b09377c
    0x7ffe3b09377c
    0x7ffe3b09377c

那怎么解释呢？

> 指针保存了地址，地址就是一个无符号整数（位数通常等于系统位数）。
> 在编译之前，指针还携带了更多的信息，那就是它的类型。
> 指针类型反映了指针指向的数据的大小（size），也就是字节数。

在这个例子中，`int`是4字节,`short`是2字节，`char`是1字节，`float`是4字节

    a = 0x40490fda

`intel x86`以`little-endian`顺序存储数据，所以在内存中

    da  <-p    p,sp,cp,fp 都指向这里
    0f
    49
    40

`*p`取出4字节，`*sp`取出2字节，`*cp`取出1字节，就出现了上述现象。

`*fp`也是4字节，二进制pattern按照一定规则被“翻译”成浮点数，参见`IEEE 754`。

> 对指针进行加减运算，相当与对地址加减size

{% highlight C %}
printf("%p\n%p\n%p\n%p\n", p, p + 1, sp + 1, cp + 1);
{% endhighlight %}

可以看到 `int *` 加了 4，`short *`加了2, `char *`加了1

    0x7ffe3b09377c
    0x7ffe3b093780
    0x7ffe3b09377e
    0x7ffe3b09377d

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

就不演示了。
