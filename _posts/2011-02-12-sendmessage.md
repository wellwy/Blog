---
layout: post_m
title:  "SendMessage、PostMessage原理"
date:   2011-02-12 16:02:29
categories: Win32
summary: 
src : 
---
{% raw %}
#SendMessage、PostMessage原理

本文讲解SendMessage、PostMessage两个函数的实现原理，分为三个步骤进行讲解，分别适合初级、中级、高级程序员进行理解，三个步骤分别为：

1. SendMessage、PostMessage的运行机制。
2. SendMessage、PostMessage的运行内幕。
3. SendMessage、PostMessage的内部实现。

注：理解这篇文章之前，必须先了解Windows的消息循环机制。
 
##SendMessage、PostMessage的运行机制
我们先来看最简单的。

SendMessage可以理解为，SendMessage函数发送消息，等待消息处理完成后，SendMessage才返回。稍微深入一点，是等待窗口处理函数返回后，SendMessage就返回了。

PostMessage可以理解为，PostMessage函数发送消息，不等待消息处理完成，立刻返回。稍微深入一点，PostMessage只管发送消息，消息有没有被送到则并不关心，只要发送了消息，便立刻返回。

对于写一般Windows程序的程序员来说，能够这样理解也就足够了。但SendMessage、PostMessage真的是一个发送消息等待、一个发送消息不等待吗？具体细节，下面第2点将会讲到。
 
##SendMessage、PostMessage的运行内幕

在写一般Windows程序时，如上第1点讲到的足以应付，其实我们可以看看MSDN来确定SendMessage、PostMessage的运行内幕。

在MSDN中，SendMessage解释如为

> The SendMessage function sends the specified message to a window or windows. It calls the window procedure for the specified window and does not return until the window procedure has processed the message.

翻译成中文为

> SendMessage函数将指定的消息发到窗口。它调用特定窗口的窗口处理函数，并且不会立即返回，直到窗口处理函数处理了这个消息。

再看看PostMessage的解释

> The PostMessage function places (posts) a message in the message queue associated with the thread that created the specified window and returns without waiting for the thread to process the message.
    
翻译成中文为

>PostMessage函数将一个消息放入与创建这个窗口的消息队列相关的线程中，并立刻返回不等待线程处理消息。

仔细看完MSDN解释，我们了解到，SendMessage的确是发送消息，然后等待处理完成返回，但发送消息的方法为直接调用消息处理函数（即WndProc函数），按照函数调用规则，肯定会等消息处理函数返回之后，SendMessage才返回。而PostMessage却没有发送消息，PostMessage是将消息放入消息队列中，然后立刻返回，至于消息何时被处理，PostMessage完全不知道，此时只有消息循环知道被PostMessage的消息何时被处理了。

至此我们拨开了一层疑云，原来SendMessage只是调用我们的消息处理函数，PostMessage只是将消息放到消息队列中。下一节将会更深入这两个函数，看看Microsoft究竟是如何实现这两个函数的。
 
##SendMessage、PostMessage的内部实现
Windows内部运行原理、机制往往是我们感兴趣的东西，而这些东西又没有被文档化，所以我们只能使用Microsoft提供的工具自己研究了。

首先，在基本Win32工程代码中，我们可以直接看到消息处理函数、消息循环，所以建立一个基本Win32工程（本篇文章使用VS2005），为了看到更多信息，我们需要进行设置，让VS2005载入Microsoft的Symbol（pdb）文件[1]。为了方便，去除了一些多余的代码，加入了两个菜单，ID分别为：`IDM_SENDMESSAGE`、`IDM_POSTMESSAGE`。如下列出经过简化后的必要的代码。

###消息循环：

	Ln000：while (GetMessage(&msg, NULL, 0, 0))
	Ln001：{
	Ln002：    TranslateMessage(&msg);
	Ln003：    DispatchMessage(&msg);
	Ln004：}
 
###消息处理函数：

	Ln100：LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
	Ln101：{
	Ln102：    int wmId, wmEvent;
	Ln103：    switch (message)
	Ln104：    {
	Ln105：    case WM_COMMAND:
	Ln106：        wmId = LOWORD(wParam);
	Ln107：        wmEvent = HIWORD(wParam);
	Ln108：        switch (wmId)
	Ln109：        {
	Ln110：        case IDM_EXIT:
	Ln111：            DestroyWindow(hWnd);
	Ln112：            break;
	Ln113：        case IDM_SENDMESSAGE:
	Ln114：            SendMessage(hWnd, WM_SENDMESSAGE, 0, 0);
	Ln115：            break;
	Ln116：        case IDM_POSTMESSAGE:
	Ln117：            PostMessage(hWnd, WM_POSTMESSAGE, 0, 0);
	Ln118：            break;
	Ln119：        default:
	Ln120：            return DefWindowProc(hWnd, message, wParam, lParam);
	Ln121：        }
	Ln122：        break;
	Ln123：
	Ln124：    case WM_SENDMESSAGE:
	Ln125：        MessageBox(hWnd, L"SendMessage", L"Prompt", MB_OK);
	Ln126：        break;
	Ln127：
	Ln128：    case WM_POSTMESSAGE:
	Ln129：        MessageBox(hWnd, L"PostMessage", L"Prompt", MB_OK);
	Ln130：        break;
	Ln131：
	Ln132：    case WM_DESTROY:
	Ln133：        PostQuitMessage(0);
	Ln134：
	Ln135：    default:
	Ln136：        return DefWindowProc(hWnd, message, wParam, lParam);
	Ln137：    }
	Ln138：    return 0;
	Ln139：}
 
下面一步步分析这两个函数的内部情况，先讨论 SendMessage。

+ 第一步，在`DispatchMessage（Ln003）`函数处下个断点，F5进行调试，当程序运行到断点后，查看 CallStack 窗口，可得如下结果：

		#003：MyProj.exe!wWinMain(HINSTANCE__ * hInstance=0x00400000, HINSTANCE__ * hPrevInstance=0x00000000, wchar_t * lpCmdLine=0x000208e0, int nCmdShow=0x00000001)  Line 49   C++
		#002：MyProj.exe!__tmainCRTStartup()  Line 589 + 0x35 bytes C
		#001：MyProj.exe!wWinMainCRTStartup()  Line 414 C
		#000：kernel32.dll!_BaseProcessStart@4()  + 0x23 bytes 

	我们可以看到，进程先调用 kernel32.dll 中的 BaseProcessStart 函数，然后调用的 Startup Code 的函数 wWinMainCRTStartup，然后调用 _tmainCRTStartup 函数，最终调用我们的 wWinMain 函数，我们的程序就运行起来了。
 
+ 第二步，去除第一步下的断点，在 `WndProc（Ln101）` 函数入口处下个断点，F5 继续运行，运行到新下的断点处，查看 CallStack 窗口，可得如下结果：
	
		#008：MyProj.exe!WndProc(HWND__ * hWnd=0x00050860, unsigned int message=0x00000101, unsigned int wParam=0x00000074, long lParam=0xc03f0001)  Line 122    C++
		#007：user32.dll!_InternalCallWinProc@20()  + 0x28 bytes   
		#006：user32.dll!_UserCallWinProcCheckWow@32()  + 0xb7 bytes   
		#005：user32.dll!_DispatchMessageWorker@8()  + 0xdc bytes  
		#004：user32.dll!_DispatchMessageW@4()  + 0xf bytes
		#003：MyProj.exe!wWinMain(HINSTANCE__ * hInstance=0x00400000, HINSTANCE__ * hPrevInstance=0x00000000, wchar_t * lpCmdLine=0x000208e0, #000：int nCmdShow=0x00000001)  Line 49 + 0xc bytes C++
		#002：MyProj.exe!__tmainCRTStartup()  Line 589 + 0x35 bytes C
		#001：MyProj.exe!wWinMainCRTStartup()  Line 414 C
		#000：kernel32.dll!_BaseProcessStart@4()  + 0x23 bytes 
    
    #000~#003 跟第一步相同，不再解释。在 #004、#005，可以看到，函数运行到 DispatchMessage 的内部了，DispatchMessageW、DispatchMessageWorker 是 user32.dll 中到处的函数，而且函数前部字符串相等，在此猜想应该是 DispatchMessage 的内部处理。#008 为我们消息处理函数，所以推想而知，#006、#007 是为了调用我们的消息处理函数而准备的代码。
 
+ 第三步，去除第二步下的断点，在Ln003、Ln114、Ln115、Ln125 处分别下一个断点，在菜单中选择对应项，使程序运行至 Ln114，F10下一步，可以看到并没有运行到 break（Ln115），直接跳到了 Ln125 处，由此可知目前 SendMessage 已经在等待了，查看 `CallStack` 窗口，可得如下结果：

		#013：MyProj.exe!WndProc(HWND__ * hWnd=0x00050860, unsigned int message=0x00000500, unsigned int wParam=0x00000000, long lParam=0x00000000)  Line 147    C++
		#012：user32.dll!_InternalCallWinProc@20()  + 0x28 bytes   
		#011：user32.dll!_UserCallWinProcCheckWow@32()  + 0xb7 bytes   
		#010：user32.dll!_SendMessageWorker@20()  + 0xc8 bytes 
		#009：user32.dll!_SendMessageW@16()  + 0x49 bytes  
		#008：MyProj.exe!WndProc(HWND__ * hWnd=0x00050860, unsigned int message=0x00000111, unsigned int wParam=0x00008003, long lParam=0x00000000)  Line 136 + 0x15 bytes   C++
		#007：user32.dll!_InternalCallWinProc@20()  + 0x28 bytes   
		#006：user32.dll!_UserCallWinProcCheckWow@32()  + 0xb7 bytes   
		#005：user32.dll!_DispatchMessageWorker@8()  + 0xdc bytes  
		#004：user32.dll!_DispatchMessageW@4()  + 0xf bytes
		#003：MyProj.exe!wWinMain(HINSTANCE__ * hInstance=0x00400000, HINSTANCE__ * hPrevInstance=0x00000000, wchar_t * lpCmdLine=0x000208e0, #000：int nCmdShow=0x00000001)  Line 49 + 0xc bytes C++
		#002：MyProj.exe!__tmainCRTStartup()  Line 589 + 0x35 bytes C
		#001：MyProj.exe!wWinMainCRTStartup()  Line 414 C
		#000：kernel32.dll!_BaseProcessStart@4()  + 0x23 bytes 

	#000~#008 跟上面的相同，不再解释。在 #009、#010，可以看到，函数调用到 SendMessage 内部了，在此猜想应该是 SendMessage 的内部处理。#011、#012 跟第二步中的 #006、#007 一样，在第二部中，这两个函数是为了调用消息处理函数而准备的代码，#013 也是我们的消息处理函数，所以此两行代码的功能相等。

	至此，我们证明了 SendMessage 的确是直接调用消息处理函数的，在消息处理函数返回前，SendMessage 等待。在所有的操作中，Ln003 断点没有去到，证明 SendMessage 不会将消息放入消息队列中（在 PostMessage 分析中，此断点将会跑到，接下来讲述）。
 
+ 第四步，F5继续运行，此时弹出对话框，点击对话框中的确定后，运行到断点 Ln115 处。查看 CallStack 窗口，可得如下结果：

		#008：MyProj.exe!WndProc(HWND__ * hWnd=0x00050860, unsigned int message=0x00000111, unsigned int wParam=0x00008003, long lParam=0x00000000)  Line 137    C++
		#007：user32.dll!_InternalCallWinProc@20()  + 0x28 bytes   
		#006：user32.dll!_UserCallWinProcCheckWow@32()  + 0xb7 bytes   
		#005：user32.dll!_DispatchMessageWorker@8()  + 0xdc bytes  
		#004：user32.dll!_DispatchMessageW@4()  + 0xf bytes
		#003：MyProj.exe!wWinMain(HINSTANCE__ * hInstance=0x00400000, HINSTANCE__ * hPrevInstance=0x00000000, wchar_t * lpCmdLine=0x000208e0, int nCmdShow=0x00000001)  Line 49 + 0xc bytes   C++
		#002：MyProj.exe!__tmainCRTStartup()  Line 589 + 0x35 bytes C
		#001：MyProj.exe!wWinMainCRTStartup()  Line 414 C
		#000：kernel32.dll!_BaseProcessStart@4()  + 0x23 bytes 
		#000~008 跟第二步的完全相同，此时 SendMessage 也已经返回，所调用的堆栈也清空了。

	至此，我们彻底拨开了 SendMessage 的疑云，了解了 SendMessage 函数的运行机制，综述为，SendMessage 内部调用 SendMessageW、SendMessageWorker 函数做内部处理，然后调用 UserCallWinProcCheckWow、InternalCallWinProc 来调用我们代码中的消息处理函数，消息处理函数完成之后，SendMessage 函数便返回了。
 

SendMessage 讨论完之后，现在讨论 PostMessage，将上面的所有断点删除，关闭调试。

+ 第一步，在DispatchMessage（Ln003）函数处下个断点，F5进行调试，此处结果跟 SendMessage 一样，不再说明。
+ 第二步，去除第一步下的断点，在 WndProc（Ln101） 函数入口处下个断点，F5 继续运行，此处结果跟 SendMessage 一样，不再说明。
+ 第三步，去除第二步下的断点，在 Ln003、Ln117、Ln118、Ln129 处分别下一个断点，在菜单中选择对应项，使程序运行至 Ln117，F10 下一步，可以看到已经运行到 break，PostMessage 函数返回了，此时 CallStack 没有变化。
+ 第四步，F5 继续运行，此时程序运行到 Ln003，CallStack 跟第一步刚起来时一样。
+ 第五步，F5 继续运行（由于有多个消息，可能要按多次），让程序运行到 Ln129，此时 CallStack 跟第二步相同，为了方便说明，再次列举如下：

		#008：MyProj.exe!WndProc(HWND__ * hWnd=0x00070874, unsigned int message=0x00000501, unsigned int wParam=0x00000000, long lParam=0x00000000)  Line 151    C++
		#007：user32.dll!_InternalCallWinProc@20()  + 0x28 bytes   
		#006：user32.dll!_UserCallWinProcCheckWow@32()  + 0xb7 bytes   
		#005：user32.dll!_DispatchMessageWorker@8()  + 0xdc bytes  
		#004：user32.dll!_DispatchMessageW@4()  + 0xf bytes
		#003：MyProj.exe!wWinMain(HINSTANCE__ * hInstance=0x00400000, HINSTANCE__ * hPrevInstance=0x00000000, wchar_t * lpCmdLine=0x000208e0, int nCmdShow=0x00000001)  Line 49 + 0xc bytes   C++
		#002：MyProj.exe!__tmainCRTStartup()  Line 589 + 0x35 bytes C
		#001：MyProj.exe!wWinMainCRTStartup()  Line 414 C
		#000：kernel32.dll!_BaseProcessStart@4()  + 0x23 bytes 

	由此可以看到，此调用是从消息循环中调用而来，DispatchMessageW、DispatchMessageWorker 是 DispatchMessage 的内部处理，UserCallWinProcCheckWow、InternalCallWinProc是为了调用我们的消息处理函数而准备的代码。
	至此，我们再次彻底拨开了 PostMessage 的疑云，了解了 PostMessage 函数的运行机制，综述为，PostMessage 将消息放入消息队列中，自己立刻返回，消息循环中的 GetMessage（PeekMessage 也可，本例中为演示）处理到我们发的消息之后，便按照普通消息处理方法进行处理。
 
 
------------------------------------
[1]关于如何设置，让VS2005载入Symbol，可以查看我写的另外一篇文章：“让Visual Studio载入Symbol（pdb）文件”，地址：http://blog.csdn.net/xt_xiaotian/archive/2010/03/16/5384111.aspx

{% endraw %}