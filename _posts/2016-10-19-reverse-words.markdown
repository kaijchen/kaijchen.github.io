---
layout: post
title:  "如何优雅的将文章中的单词反转"
date:   2016-10-19 11:53:58 +0800
categories: Coding
---

## 问题描述

一道数据结构上机题

> 反转输入的每个数字，但是要保持格式不变（换行空格什么的）

### 思路

把数字看作广义的字符串，也就是本文标题

1. 空白字符保持不变，原样输出
2. 读取一串字符直到空白，反向输出
3. 重复1 2，直到文件结束

算法很简单，但是想优雅地实现还是不容易的。  

#### 何谓优雅呢？

* 设计简洁，由很简单的子程序构成，一次做好一件事
* 适用性广，对数据没多少要求（每个单词可以很长很长）

### 实现

逆序输出一个字符串到文件`fp`

{% highlight C %}
void printr(const char *s, FILE *fp)
{
	int i;

	for (i = strlen(s) - 1; i >= 0; i--)
		putc(s[i], fp);
}
{% endhighlight %}

---

处理空白字符，从`in`读取，输出到`out`

{% highlight C %}
int despace(FILE *in, FILE *out)
{
	int c;

	/* 处理接下来的空字符，直到文件结束或读到非空字符 */
	while ((c = getc(in)) != EOF && isspace(c))
		putc(c, out);

	/* 如果读到了非空字符，要放回去 */
	if (c != EOF)
		ungetc(c, in);

	/* 返回最终状态作文件结束判断依据 */
	return c;
}
{% endhighlight %}

---

从文件`fp`读取一个单词并返回，  
这个稍微长一点，不过主要是在处理异常

{% highlight C %}
char *nextword(FILE *fp)
{
	/* 
	 * static 修饰的数据在.bss 或者.data段，
	 * 而不是运行时栈，
	 * 所以在函数外面依然存在。
	 */
	static char *buf = NULL;
	static int size = 0;
	void *nbuf;
	int i, c;


	/* 第一次调用时初始化 buf 和 size */
	if (buf == NULL) {
		size = 1;
		buf = malloc(size * sizeof(*buf));
		if (buf == NULL) {
			/* 内存不够了 */
			reset(&size, &buf);
			return NULL;
		}
	}

	/* 处理下一个字符，如果文件结束或它是空白，就退出 */
	for (i = 0; (c = getc(fp)) != EOF && !isspace(c); i++) {
		if (i + 1 >= size) {
			/* buf 要被填满了，扩充 buf 的大小为两倍 */
			size *= 2;
			nbuf = realloc(buf, size * sizeof(*buf));
			if (nbuf == NULL) {
				/* 内存不够了 */
				reset(&size, &buf);
				return NULL;
			}
			buf = nbuf;
		}
		/* 一切 OK，把字符写到 buf 里 */
		buf[i] = c;
	}
	/* 加上字符串结束标记 */
	buf[i] = '\0';

	/* 把空字符放回去 */
	if (c != EOF)
		ungetc(c, fp);
	return buf;
}
{% endhighlight %}

同时，所用到的reset函数
{% highlight C %}
void reset(int *p, char **q)
{
	free(*q);
	*q = NULL;
	*p = 0;
}
{% endhighlight %}

---

因为前面做了很多工作，我们的`main`函数非常简单

{% highlight C %}
int main()
{
	char *word;

	/* 无限循环 */
	for (;;) {
		/*
		 * 前面写的函数派上了用场
                 * 所以这里就很简单了 :)
                 */
		if (despace(stdin, stdout) == EOF)
			break;

		if ((word = nextword(stdin)) == NULL)
			fprintf(stderr, "out of memory\n");
		else
			printr(word, stdout);
	}
	return 0;
}
{% endhighlight %}

### 完整的源代码

{% highlight C %}
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* 逆序输出一个字符串到文件fp */
void printr(const char *s, FILE *fp)
{
	int i;

	for (i = strlen(s) - 1; i >= 0; i--)
		putc(s[i], fp);
}

/* 出问题了重置，在 nextword 里用到 */
void reset(int *p, char **q)
{
	free(*q);
	*q = NULL;
	*p = 0;
}

/*
 * 从文件 fp 读取一个单词并返回
 * 单词的定义是用空白字符隔开的连续字符片段
 * 空白字符由 ctype.h 中 isspace() 定义。
 * 如需修改或保存获得的单词，需要复制一份。
 */
char *nextword(FILE *fp)
{
	/* 
	 * static 修饰的数据在.bss 或者.data段，
	 * 而不是运行时栈，
	 * 所以在函数外面依然存在。
	 */
	static char *buf = NULL;
	static int size = 0;
	void *nbuf;
	int i, c;


	/* 第一次调用时初始化 buf 和 size */
	if (buf == NULL) {
		size = 1;
		buf = malloc(size * sizeof(*buf));
		if (buf == NULL) {
			/* 内存不够了 */
			reset(&size, &buf);
			return NULL;
		}
	}

	/* 处理下一个字符，如果文件结束或它是空白，就退出 */
	for (i = 0; (c = getc(fp)) != EOF && !isspace(c); i++) {
		if (i + 1 >= size) {
			/* buf 要被填满了，扩充 buf 的大小为两倍 */
			size *= 2;
			nbuf = realloc(buf, size * sizeof(*buf));
			if (nbuf == NULL) {
				/* 内存不够了 */
				reset(&size, &buf);
				return NULL;
			}
			buf = nbuf;
		}
		/* 一切 OK，把字符写到 buf 里 */
		buf[i] = c;
	}
	/* 加上字符串结束标记 */
	buf[i] = '\0';

	/* 把空字符放回去 */
	if (c != EOF)
		ungetc(c, fp);
	return buf;
}

/* 处理空白字符，从 in 读取，输出到 out */
int despace(FILE *in, FILE *out)
{
	int c;

	/* 处理接下来的空字符，直到文件结束或读到非空字符 */
	while ((c = getc(in)) != EOF && isspace(c))
		putc(c, out);

	/* 如果读到了非空字符，要放回去 */
	if (c != EOF)
		ungetc(c, in);

	/* 返回最终状态作文件结束判断依据 */
	return c;
}

int main()
{
	char *word;

	/* 无限循环 */
	for (;;) {
		/*
		 * 前面写的函数派上了用场
                 * 所以这里就很简单了 :)
                 */
		if (despace(stdin, stdout) == EOF)
			break;

		if ((word = nextword(stdin)) == NULL)
			fprintf(stderr, "out of memory\n");
		else
			printr(word, stdout);
	}
	return 0;
}
{% endhighlight %}
