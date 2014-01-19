---
layout: post_m
title:  "_beginthreadex相关的东东"
date:   2011-02-12 16:56:54
categories: win32
description:  
src :  
---

## 关于`_endthread`和`_endthreadex`

谈到Handle的问题，`_beginthread`的对应函数`_endthread`自动的调用了`CloseHandle`，而`_beginthreadex`的对应函数`_endthreadex`则没有，所以`CloseHandle`无论如何都是要调用的不过`_endthread`可以帮你执行自己不必写，其他两种就需要自己写！(Jeffrey   Richter强烈推荐尽量不用显式的终止函数，用自然退出的方式，自然退出当然就一定要自己写CloseHandle) 

 

>You can call `_endthread` or `_endthreadex` explicitly to terminate a thread; however, `_endthread` or` _endthreadex` is called automatically when the thread returns from the routine passed as a parameter. Terminating a thread with a call to `endthread` or `_endthreadex` helps to ensure proper recovery of resources allocated for the thread.

##关于创建多线程

在Windows下面，比较常见的多线程创建函数是`CreateThread`(Windows自带的)和`_beginthread`、`_beginthreadex`（C运行库的）。

强烈推荐只用`_beginthreadex`，参数上面与`CreateThread`基本一样，比`CreateThread`好的就是，它给每个线程维护一个`tiddata`数据块，这对于多线程环境非常重要。《Windows核心编程》里也是这么说的。

用`_beginthreadex`还有个好处是，线程结束后，不会自己执行`CloseHandle`函数，这样的话，对于WaitFor系列函数的调用就比较方便了。`_beginthread`在线程结束的时候是会自己`CloseHandle`。

## `_beginthread()`存在几个缺陷
如下： 

1. 创建时不能将线程挂起。 
2.     _beginthread产生出来的线程会首先关闭自己的handle，这样做是为了隐藏win32的实现细节，因此_beginthread()传回的参数可能在当时是不可用的。因此，没有这个handle，你就无法等待他结束，无法改变其参数，无法获得结束代码。 


##怎么知道_beginthreadex开始的线程被结束

    HANDLE hThread;
	hThread = (HANDLE)_beginthreadex( NULL, 0, &SecondThreadFunc, NULL, 0, &threadID );

    WaitForSingleObject( hThread, INFINITE );
 
以上内容来自MSDN