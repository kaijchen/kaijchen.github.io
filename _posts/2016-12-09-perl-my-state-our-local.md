---
layout: post
title:  "perl 中的 my state our local"
date:   2016-12-09 15:29:05 +0800
categories: notes
---

	my	局部变量
	state	相当于 C 中的 static
	our	全局变量
	local	临时覆盖全局变量

一个 `local` 的示例：

```perl
#!/usr/bin/env perl
use v5.24;

our $fire = "hello";

sub foo {
	local $fire = "world";
	say "foo: $fire";	# "world"
	&bar;			# bar 中 $fire 是 foo 的 local $fire
}

sub bar {
	say "bar: $fire";
	$fire = "qwerty";
}

say $fire;	# our $fire 是 "hello"
&foo;		# foo 调用了 bar
say $fire;	# our $fire 还是 "hello"
&bar;		# bar 中的 $fire 是 our $fire
say $fire;	# our $fire 被修改成 "qwerty"
```
