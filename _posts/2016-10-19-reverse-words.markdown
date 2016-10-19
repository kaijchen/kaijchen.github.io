---
layout: post
title:  "如何优雅的将文章中的单词反转"
date:   2016-10-19 11:53:58 +0800
categories: Coding
---

## 问题描述

> 反转输入的每个数字，但是要保持格式不变（换行空格什么的）

### 引申一下，把数字看作广义的字符串，就成了标题了

一道数据结构上机题，算法很简单，想优雅地实现还是不容易的。  
算是练习一下 `The Practice of Programming` 里的学到的知识吧。

### 何谓优雅呢？

设计简洁，由很简单的子程序构成，一次做好一件事；  
适用性广，对数据没多少要求（每个单词可以很长很长）

> Talk is cheap, show me the code.
> 废话少说，放码过来。

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
		putchar(s[i], fp);
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

int main()
{
	char *word;
	int c;

	for (;;) {
		/* 处理接下来的空字符，直到文件结束或读到非空字符 */
		while ((c = getchar()) != EOF && isspace(c))
			putchar(c);
		/*
		 * 文件结束就可以走人了；
		 * 但是如果读到了非空字符，要放回去
		 */
		if (c != EOF)
			ungetc(c, stdin);
		else
			break;

		/*
		 * 前面写的函数派上了用场
                 * 所以这里就很简单了 :)
                 */
		if ((word = nextword(stdin)) == NULL)
			fprintf(stderr, "out of memory\n");
		else
			printr(word);
	}
	return 0;
}
{% endhighlight %}

