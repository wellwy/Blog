---
layout: post_m
title:  "C++ 续行符"
date:   2011-02-12 15:26:57
categories: Win32
summary: 
src : 
---

#C++ 续行符

##Question
VC中的单斜杠"\"用在行末尾是什么意思？

例如：

	#define CODE_WORK  \
	index_check=toIndex(check);\
	if(next==target) return checkstep[index_check]+1;\
	index_next=toIndex(next);\ 

##Answer

这玩意儿有个正式的名称叫做续行符,在普通代码行后面加不加都一样(VC是自动判断续行的),但是在宏定义里面就特别有用,因为宏定义规定必须用一行完成:

    #define SomeFun(x, a, b) if(x)x=a+b;else x=a-b;

这一行定义是没有问题的,但是这样代码很不容易被理解,以后维护起来麻烦,如果写成:

	#define SomeFun(x, a, b)
	    if (x)
	        x = a + b;
	    else
	        x = a - b;

这样理解是好理解了,但是编译器会出错,因为它会认为#define SomeFun(x, a, b)是完整的一行,if (x)以及后面的语句与#define SomeFun(x, a, b)没有关系.这时候我们就必须使用这样的写法:

	#define SomeFun(x, a, b)\
	    if (x)\
	        x = a + b;\
	    else\
	        x = a - b;
	        
注意:最后一行不要加续行符啊.VC的预处理器在编译之前会自动将\与换行回车去掉,这样一来既不影响阅读,又不影响逻辑,皆大欢喜. 