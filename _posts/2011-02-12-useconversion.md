---
layout: post_m
title:  "慎用USES_CONVERSION"
date:   2011-02-12 15:25:56
categories: Win32
description: 
src : http://blog.csdn.net/vcleaner/article/details/1731171
---

#慎用USES_CONVERSION

`USES_CONVERSION`是ATL中的一个 宏定义。用于编码转换(用的比较多的是`CString向LPCWSTR`转换)。在ATL下使用要包含头文件`#include "atlconv.h"`

使用`USES_CONVERSION`一定要小心，它们从堆栈上分配内存，直到调用它的函数返回，该内存不会被释放。如果在一个循环中，这个宏被反复调用几万次，将不可避免的产生`stackoverflow`。

#解决方法

将用到该宏的语句独立封装成一个函数，这样就可以无限次调用了。

例：

	void fn()
	{
	    while (true)
	    {
	        USES_CONVERSION;
	        DoSomething(A2W("Something"));
	    }
	}

改成------------------------------->

	void fn2()
	{
	    USES_CONVERSION;
	    DoSomething(A2W("Something"));
	}
	void fn()
	{
	    while(true)
	    {
	        fn2();
	    }
	}