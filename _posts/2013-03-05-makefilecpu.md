---
layout: post_m
title:  "Makefile和automake中判断CPU位数"
date:   2013-03-05 09:49:33
categories: Makefile
description:  
src :  
---

Makefile中:
	

    cpu_bit=$(shell getconf LONG_BIT)
	ifeq ($(cpu_bit),64)
	    MY_CXXFLAGS=
	else
	    MY_CXXFLAGS=-march=pentium4
	endif

	x::
        @echo $(cpu_bit)
        @echo $(MY_CXXFLAGS)

automake中要稍复杂些:

首先要在configure.ac文件中添加一行:

    AM_CONDITIONAL(bit_32,test "x`getconf LONG_BIT`"="x32")

然后再在Makefile.am文件中添加:

	if bit_32
	    MY_CXXFLAGS=-march=pentium4
	else
	    MY_CXXFLAGS=
	endif

这样就可以了.