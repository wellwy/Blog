---
layout: post_m
title:  "线程中CreateEvent和SetEvent及WaitForSingleObject的用法"
date:   2011-02-12 15:51:29
categories: Win32
description: 
src : 
---

#功能描述
+ `CreateEvent`:创建或打开一个命名的或无名的事件对象.

	EVENT有两种状态：**发信号**，**不发信号**。 
+ `SetEvent/ResetEvent`分别将EVENT置为这两种状态分别是发信号与不发信号。 
+ `WaitForSingleObject()`等待，直到参数所指定的OBJECT成为发信号状态时才返回，OBJECT可以是EVENT，也可以是其它内核对象。

当你创建一个线程时，其实那个线程是一个循环，不像上面那样只运行一次的。

这样就带来了一个问题，在那个死循环里要找到合适的条件退出那个死循环，那么是怎么样实现它的呢？在Windows里往往是采用事件的方式，当然还可以采用其它的方式。在这里先介绍采用事件的方式来通知从线程运行函数退出来，它的实现原理是这样，在那个死循环里不断地使用`WaitForSingleObject`函数来检查事件是否满足，如果满足就退出线程，不满足就继续运行。当在线程里运行阻塞的函数时，就需要在退出线程时，先要把阻塞状态变成非阻塞状态，比如使用一个线程去接收网络数据，同时使用阻塞的SOCKET时，那么要先关闭SOCKET，再发送事件信号，才可以退出线程的。

当然我感觉重要应用方面还是用来锁定，实现所谓的pv功能。

#函数功能，参数等
##CreateEvent
 
###函数功能描述
创建或打开一个命名的或无名的事件对象
###函数原型
	HANDLE CreateEvent(
	  LPSECURITY_ATTRIBUTES lpEventAttributes,   // 安全属性
	  BOOL bManualReset,   // 复位方式
	  BOOL bInitialState,   // 初始状态
	  LPCTSTR lpName   // 对象名称
	);
###参数
- **lpEventAttributes**：

	+ *[输入]*一个指向`SECURITY_ATTRIBUTES`结构的指针，确定返回的句柄是否可被子进程继承。如果`lpEventAttributes`是NULL，此句柄不能被继承。
	+ **Windows NT/2000**：`lpEventAttributes`的结构中的成员为新的事件指定了一个安全符。如果`lpEventAttributes`是NULL，事件将获得一个默认的安全符。
- **bManualReset**：

	+ *[输入]*指定将事件对象创建成手动复原还是自动复原。如果是TRUE，那么必须用`ResetEvent`函数来手工将事件的状态复原到无信号状态。如果设置为FALSE，当事件被一个等待线程释放以后，系统将会自动将事件状态复原为无信号状态。
	
- **bInitialState**：
	      
	+ *[输入]*指定事件对象的初始状态。如果为TRUE，初始状态为有信号状态；否则为无信号状态。

- **lpName**：
	+ *[输入]*指定事件的对象的名称，是一个以0结束的字符串指针。名称的字符格式限定在`MAX_PATH`之内。名字是对大小写敏感的。

	+ 如果lpName指定的名字，与一个存在的命名的事件对象的名称相同，函数将请求`EVENT_ALL_ACCESS`来访问存在的对象。这时候，由于`bManualReset`和`bInitialState`参数已经在创建事件的进程中设置，这两个参数将被忽略。如果`lpEventAttributes`是参数不是NULL，它将确定此句柄是否可以被继承，但是其安全描述符成员将被忽略。
      
	+ 如果lpName为NULL，将创建一个无名的事件对象。
    + 如果lpName的和一个存在的信号、互斥、等待计时器、作业或者是文件映射对象名称相同，函数将会失败，在GetLastError函数中将返回ERROR_INVALID_HANDLE。造成这种现象的原因是这些对象共享同一个命名空间。
    + *终端服务(Terminal Services)*：名称中可以加入`Global\`或是`Local\`的前缀，这样可以明确的将对象创建在全局的或事务的命名空间。名称的其它部分除了反斜杠(\)，可以使用任意字符。详细内容可参考Kernel Object Name Spaces。
    + Windows 2000：在Windows 2000系统中，没有终端服务运行，"Global\"和"Local\"前缀将被忽略。名称的其它部分除了反斜杠(\)，可以使用任意字符。
    + Windows NT 4.0以及早期版本, Windows 95/98：名称中除了反斜杠(\)，可以使用任意字符。
###返回值

+ 如果函数调用成功，函数返回事件对象的句柄。如果对于命名的对象，在函数调用前已经被创建，函数将返回存在的事件对象的句柄，而且在GetLastError函数中返回ERROR_ALREADY_EXISTS。

+ 如果函数失败，函数返回值为NULL，如果需要获得详细的错误信息，需要调用GetLastError。

###备注：
      
+ 调用CreateEvent函数返回的句柄，该句柄具有EVENT_ALL_ACCESS权限去访问新的事件对象，同时它可以在任何有此事件对象句柄的函数中使用。
      
+ 在调用的过程中，所有线程都可以在一个等待函数中指定事件对象句柄。当指定的对象的状态被置为有信号状态时，单对象等待函数将返回。
      
+ 对于多对象等待函数，可以指定为任意或所有指定的对象被置为有信号状态。当等待函数返回时，等待线程将被释放去继续运行。
      
+ 初始状态在bInitialState参数中进行设置。使用SetEvent函数将事件对象的状态置为有信号状态。使用ResetEvent函数将事件对象的状态置为无信号状态。
      
	+ 当一个手动复原的事件对象的状态被置为有信号状态时，该对象状态将一直保持有信号状态，直至明确调用ResetEvent函数将其置为无符号状态。
	      
	+ 当事件的对象被置为有信号状态时，任意数量的等待中线程，以及随后开始等待的线程均会被释放。
	      
	+ 当一个自动复原的事件对象的状态被置为有信号状态时，该对象状态将一直保持有信号状态，直至一个等待线程被释放;系统将自动将此函数置为无符号状态。如果没有等待线程正在等待，事件对象的状态将保持有信号状态。
      
+ 多个进程可持有同一个事件对象的多个句柄，可以通过使用此对象来实现进程间的同步。下面的对象共享机制是可行的：
      
+ 在CreateEvent函数中，lpEventAttributes参数指定句柄可被继承时，通过CreateProcess函数创建的子进程继承的事件对象句柄。
      
一个进程可以在DuplicateHandle函数中指定事件对象句柄，从而获得一个复制的句柄，此句柄可以被其它进程使用。
      
一个进程可以在OpenEvent或CreateEvent函数中指定一个名字，从而获得一个有名的事件对象句柄。
      
使用CloseHandle函数关闭句柄。当进程停止时，系统将自动关闭句柄。当最后一个句柄被关闭后，事件对象将被销毁。

###使用环境：
      
**Windows NT/2000**：需要3.1或更高版本

**Windows 95/98**：需要Windows 95或更高版本

- 头文件：定义在Winbase.h；需要包含 Windows.h。
- 导入库：user32.lib

**Unicode**：在Windows NT/2000中，以 Unicode 和 ANSI 执行

> 一个Event被创建以后,可以用OpenEvent()API来获得它的Handle,用CloseHandle()来关闭它,用SetEvent()或PulseEvent()来设置它使其有信号,用ResetEvent()来使其无信号,用WaitForSingleObject()或WaitForMultipleObjects()来等待其变为有信号.   

> PulseEvent()是一个比较有意思的使用方法,正如这个API的名字,它使一个Event对象的状态发生一次脉冲变化,从无信号变成有信号再变成无信号,而整个操作是原子的.对自动复位的Event对象,它仅释放第一个等到该事件的thread（如果有),而对于人工复位的Event对象,它释放所有等待的thread.  

##WaitForSingleObject
###函数原型

	DWORD WaitForSingleObject( 
	   HANDLE hHandle, 
	   DWORD dwMilliseconds 
	); 

参数hHandle是一个事件的句柄，第二个参数dwMilliseconds是时间间隔。如果时间是有信号状态返回`WAIT_OBJECT_0`，如果时间超过dwMilliseconds值但时间事件还是无信号状态则返回`WAIT_TIMEOUT`。

hHandle可以是下列对象的句柄：

+ Change notification 
+ Console input 
+ Event 
+ Job 
+ Memory resource notification 
+ Mutex 
+ Process 
+ Semaphore 
+ Thread 
+ Waitable timer 

`WaitForSingleObject`函数用来检测hHandle事件的信号状态，当函数的执行时间超过`dwMilliseconds`就返回，但如果参数`dwMilliseconds`为`INFINITE`时函数将直到相应时间事件变成有信号状态才返回，否则就一直等待下去，直到`WaitForSingleObject`有返回直才执行后面的代码。在这里举个例子：

+ 先创建一个全局Event对象g_event:

		CEvent g_event;

+ 在程序中可以通过调用`CEvent::SetEvent`设置事件为有信号状态。
	
	下面是一个线程函数MyThreadPro()

		UINT CFlushDlg::MyThreadProc( LPVOID pParam ) 
		{ 
		    WaitForSingleObject(g_event,INFINITE); 
		    For(;;) 
		       { 
		      …………. 
		       } 
		    return 0; 
		} 

	在这个线程函数中只有设置g_event为有信号状态时才执行下面的for循环，因为g_event是全局变量，所以我们可以在别的线程中通过g_event. SetEvent控制这个线程。

	还有一种用法就是我们可以通过WaitForSingleObject函数来间隔的执行一个线程函数的函数体

		UINT CFlushDlg::MyThreadProc( LPVOID pParam ) 
		{ 
		    while(WaitForSingleObject(g_event,MT_INTERVAL)!=WAIT_OBJECT_0) 
		    { 
		      ……………… 
		    } 
		    return 0; 
		} 
	
	在这个线程函数中可以可以通过设置`MT_INTERVAL`来控制这个线程的函数体多久执行一次，当事件为无信号状态时函数体隔`MT_INTERVAL`执行一次，当设置事件为有信号状态时，线程就执行完毕了