---
layout: post
title:  "KMP模式匹配"
date:   2016-06-07 16:16:41 +0800
categories: Algorithm
---

关于模式匹配，就是在一个字符串里查找字串。

以前只会`tire`构造有限自动机，觉得直观好理解。
突然发现，KMP不就是树降成链表的有限自动机吗？

直接贴代码吧，一会儿再解释（来不及解释了，快上车 =_= !）
{% highlight C %}
#include <string.h>

char *strpos(const char *haystack, const char *needle)
{
	static int fail[1000] = {-1};
	int l = strlen(needle) - 1;
	int j;

	/* pre-process */
	for (int i = 1; i <= l; i++) {
		j = fail[i - 1];
		while (j != -1 && needle[j + 1] != needle[i])
		     j = fail[j];
		if (needle[j + 1] == needle[i])
			fail[i] = j + 1;
		else
			fail[i] = -1;
	}

	/* find first match */
	j = -1;
	for (const char *p = haystack; *p; p++) {
		while (j != -1 && *p != needle[j + 1])
			j = fail[j];
		if (*p == needle[j + 1])
			j++;
		if (j == l)
			return (char *)p - l;
	}
	return NULL;
}
{% endhighlight %}