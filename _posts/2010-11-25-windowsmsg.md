---
layout: post_m
title:  "Windows消息拦截技术的应用"
date:  2010-11-25 09:23:57
categories: win32
description:  
src :  
---

##前　言
众所周知，Windows程式的运行是依靠发生的事件来驱动。换句话说，程式不断等待一个消息的发生，然后对这个消息的类型进行判断，再做适当的处理。处理完此次消息后又回到等待状态。从上面对Windows程式运行机制的分析不难发现，消息在用户与程式之间进行交流时起了一种中间“语言”的作用。在程式中接收和处理消息的主角是窗口，它通过消息泵接收消息，再通过一个窗口过程对消息进行相应的处理。

消息拦截的实现是在窗口过程处理消息之前拦截到消息并做相关处理后再传送给原窗口过程。通常情况下，程序员可以在窗口过程中处理接收到的消息，这就要求窗口过程必须在开发程序时完成，但是在一些应用中常常需要获取和处理另外应用程序或其它单元模块中的消息，实现此类功能的技术也就本文要讨论的主题――消息拦截技术。
##理解Windows消息机制
在深入探讨消息拦截技术实现原理之前，让我们先来温习一下Windows消息机制原理知识。

###消息的产生
消息作为程序与外界交流的“语言”，它的产生自然来自外界，但这里所说的外界，不只是简单的指程序之外或软件系统之外，而是泛指消息处理模块之外的模块、Windows系统、其它应用程序以及硬件等。通常根据消息产生的方式将其分为两大类，即硬件消息和软件消息。硬件消息，常指由硬件装置所产生的事件（如鼠标或键盘被按下），放在系统消息队列（System Queue）中，再由系统消息处理机构将消息发送给应用程序消息队列中。软件消息，常指由Windows系统或其它应用程序发送的信息，它直接放入应用程序消息队列（Application Queue）中，再由应用程序消息处理机构将消息传递给相应的窗口。
### 消息的组成
一个消息由一个消息名称（UINT），和两个参数（WPARAM，LPARAM）。当用户进行了输入或是窗口的状态发生改变时系统都会发送消息到某一个窗口。例如当菜单转中之后会有WM_COMMAND消息发送，WPARAM的高字中（HIWORD(wParam)）是命令的ID号，对菜单来讲就是菜单ID。当然用户也可以定义自己的消息名称，也可以利用自定义消息来发送通知和传送数据。
###消息的接收者
一个消息必须由一个窗口接收。在窗口过程（WNDPROC）中可以对消息进行分析，对应用程序要求处理的消息进行相应的处理工作，对于那么不需要应用程序处理的消息可简单的调用缺省处理。例如你希望对菜单选择进行处理那么你可以定义对WM_COMMAND进行处理的代码，如果希望在窗口中进行图形输出就必须对WM_PAINT进行处理。
###消息的处理
窗口接收到发送给自己的消息后，将消息结构作为参数调用窗口过程对消息进行相应的处理。可以将窗口过程看作消息处理代码的集合，窗口过程函数的原型为：

    long FAR PASCAL WndProc(HWND hWnd,WORD message,WORD wParam,LONG lParam);

其中，hWnd为窗口句柄，message为消息名称，wParam,lParam为两个参数。

在Windows中，应用程序不直接调用任何窗口函数，而是等待Windows调用窗口函数，请求完成任务或返回信息。为保证Windows调用这个窗口函数，这个函数必须先向Windows登记，然后在Windows实施相应操作时回调，所以窗口函数又称为回调函数。WndProc是一个主回调函数，Windows至少有一个回调函数。它是在应用程序进行窗口类注册时向Windows登记的。
 
##利用钩子（Hook）拦截消息
###何为钩子（Hook）

钩子（Hook）机制允许应用程序截获处理window消息或特定事件。与DOS中断截获处理机制有类似之处。钩子是Windows消息处理机制的一个平台，应用程序可以在上面设置子程以监视指定窗口的某种消息，而且所监视的窗口可以是其他进程所创建的。当消息到达后，钩子可以在目标窗口处理函数之前处理它并且可以阻止消息的传递。每一个钩子都有一个与之相关联的指针列表，称之为钩子链表，该链表中的指针指向这个钩子的各个处理子程。钩子的种类很多，每种钩子可以拦截并处理相应种类的消息。当钩子所监视的消息出现时，Windows调用链表中的第一个钩子子程，第一个过程完成后将消息传递链表中的下一个钩子子程，直至链表中所有钩子子程都执行完成（注意：如果在其中有一个钩子在执行完成前不执行消息传递，其后面的钩子过程和原窗口过程都不会再接收到消息。）后将消息返回给窗口过程。
###钩子子程函数
钩子子程是一个应用程序定义的回调函数。用以监视系统或某一特定类型的事件，这些事件可以是与某一特定线程关联的，也可以是系统中所有线程的事件。其函数原型为：
 
	LRESULT CALLBACK HookProc  (  int nCode, WPARAM wParam, LPARAM lParam );
 
其中，nCode参数是Hook代码，Hook子程使用这个参数来确定任务。这个参数的值依赖于Hook类型，每一种Hook都有自己的Hook代码特征字符集。 Windows系统提供了多种类型的钩子，每一种类型的Hook可以使应用程序能够监视不同类型的系统消息处理机制。

wParam和lParam参数的值依赖于Hook代码，但是它们的典型值是包含了关于发送或者接收消息的信息。

###钩子的安装与卸载

钩子的安装是通过SDK API SetWindowsHookEx()来实现的，它将钩子子程安装到系统钩子链表中。其函数原型
 
	HHOOK SetWindowsHookEx( int idHook,HOOKPROC lpfn,HINSTANCE hMod, DWORD dwThreadId );
 
其中

- `idHook`是指钩子的类型。表一中列出部分钩子的类型及其说明。

	|类型|说明
	|----|----|
	|WH_CALLWNDPROC| 系统在消息发送到接收窗口过程之前调用此子程
	|WH_CALLWNDPROCRET Hooks|在窗口过程处理完消息之后调用此子程
	|WH_GETMESSAGE|监视从GetMessage / PeekMessage函数返回的消息
	|WH_KEYBOARD|监视输入到消息队列中的键盘消息
	|WH_MOUSE|监视输入到消息队列中的鼠标消息
	 
	限于篇幅，其它消息类型就不一一列出了。有关内容可参见MSDN。
 
 
- `lpfn`是指钩子子程的地址指针。如果`dwThreadId`参数为0 或是一个由别的进程创建的线程的标识，`lpfn`必 须指向DLL中的钩子子程。除此以外，`lpfn`可以指向当前进程的一段钩子子程代码。
- `hMod`是指应用程序实例的句柄。标识包含lpfn所指的子程的DLL。如果dwThreadId 标识当前进程创建的一个线程，而且子程代码位于当前进程，hMod必须为NULL。
- `dwThreadId`是指与安装的钩子子程相关联的线程的标识符，如果为0，钩子子程与所有的线程关联。

函数成功则返回钩子的句柄，失败返回NULL。
 
钩子在使用完之后需要用`UnHookWindowsHookEx()`卸载，否则会造成麻烦。卸载钩子比较简单，`UnHookWindowsHookEx()`只有一个参数。函数原型如下：

	UnHookWindowsHookEx  ( HHOOK hhk  )

其中，参数`hhk`是`SetWindowsHookEx()`函数返回钩子句柄，所以设计程序时一定要保存好这个句柄，以便卸载时使用。函数成功返回TRUE，否则返回FALSE。
 
###系统钩子与线程钩子
Windows系统根据钩子监视事件的范围将钩子分为系统钩子（全局钩子）和线程钩子（局部钩子）两种。由SetWindowsHookEx()函数的最后一个参数决定了此钩子是系统钩子还是线程钩子。线程勾子用于监视指定线程的事件消息。线程勾子一般在当前线程或者当前线程派生的线程内。 系统勾子监视系统中的所有线程的事件消息。因为系统勾子会影响系统中所有的应用程序，所以勾子函数必须放在独立的动态链接库(DLL) 中。系统自动将包含"钩子回调函数"的DLL映射到受钩子函数影响的所有进程的地址空间中，即将这个DLL注入了那些进程。
 
### 钩子的实现
本文的实例实现拦截记事本(NotePad.exe)程序的WM_CHAR消息的功能。如读者想实现其它功能，可直接在钩子子程函数中加入代码。

1. 选择MFC AppWizard(DLL)创建项目NotePadhook并选择MFC Extension DLL（共享MFC拷贝）类型。
2. 创建NotePadHook.h文件，在其中建立钩子类：
	
		class AFX_EXT_CLASS CNotePadHook:public CObject 
		{　
			ublic:
			CNotePadHook(); //钩子类的构造函数
			~CNotePadHook(); //钩子类的析构函数
			BOOL StartHook(HWND hWnd);  //安装钩子函数
			BOOL StopHook(); 卸载钩子函数
		};
3. 在NotePadHook.cpp中加入＃i nclude “NotePadHook.h”。
4. 在NotePadHook.cpp中加入共享数据段：
	
		#pragma data_seg("sharedata")  //共享数据段，段内的变量可被钩子所有实例共享。
		HHOOK glhHook=NULL;  //钩子句柄。
		HINSTANCE glhInstance=NULL;  //DLL实例句柄。
		#pragma data_seg() 

5. 仅定义一个数据段还不能达到共享数据的目的，还要告诉编译器该段的属性。要在.DEF文件中设置段的属性，打开.DEF文件加入如下代码：
	
		SETCTIONS
		sharedata READ WRITE SHARED

6. 在主文件NotePadHook.cpp的DllMain函数中加入保存DLL实例句柄：
	
		DllMain(HINSTANCE hInstance, DWORD dwReason, LPVOID lpReserved)
		{
		      //如果使用lpReserved参数则删除下面这行
		      UNREFERENCED_PARAMETER(lpReserved);
		 
		      if (dwReason == DLL_PROCESS_ATTACH)
		      {
		            TRACE0("NOtePadHOOK.DLL Initializing!\n");
		             //扩展DLL仅初始化一次
		           if (!AfxInitExtensionModule(NotePadHookDLL, hInstance))
		                 return 0;
		            new CDynLinkLibrary(NotePadHookDLL);
					//把DLL加入动态MFC类库中
		            glhInstance = hInstance;
		       		//插入保存DLL实例句柄
		      }
		      else if (dwReason == DLL_PROCESS_DETACH)
		      {
		            TRACE0("NotePadHOOK.DLL Terminating!\n");
		           //终止这个链接库前调用它
		            AfxTermExtensionModule(NotePadHookDLL);
		      }
		      return 1;  
		}

7. 类CNotePadHook的成员函数的具体实现：
	
		CNotePadHook::CNotePadHook(){}
		CNotePadHook::~CNotePadHook(){ StopHook(); }
		BOOL CNotePadHook::StartHook(HWND hWnd) //安装钩子。
		{
		  BOOL bResult=FALSE;
		glhHook=SetWindowsHookEx(WH_CALLWNDPROC,NotePadProc,glhInstance,0);
		  if(glhHook!=NULL) bResult=TRUE;
		return bResult;
		}
		CNotePadHook::StopHook()
		{
		   BOOL bResult = FALSE;
		   if(glhHook){
		      bResult=UnhookWindowsHookEx(glhHook);
		   if(bResult)  glhHook=NULL;
		   return bResult;
		}

8. 钩子子程的实现：
	
		LRESULT WINAPI NotePadProc(int nCode,WPARAM wparam,LPARAM lparam)
		{
			PCWPSTRUCT pcs = NULL;
			pcs = (PCWPSTRUCT)lParam;
			if( pcs && pcs->hwnd!=NULL )
			{
				TCHAR szClass[256];
		     	GetClassName(pcs->hwnd ,szClass,255);//获得拦截的窗口类名。
		    	if( strcmp(szClass,"Notepad")==0)
		    	{
		     	if( pcs->message == WM_CHAR )
		     	{
		           	AfxMessageBox("HOOK NOTEPAD WM_CHAR OK!!!");
		    	}
			}
		    return CallNextHookEx(glhHook,nCode,wParam,lParam);//继续传递消息。
		}
 

9. 编译项目生成NotePadHook.dll。
 	虽然已经完成了钩子类，但还不能实现钩子功能。我们必须写一个程序来启动钩子，将钩子DLL注入其它程序的内存空间并将钩子加入到系统钩子链表中。由于限于篇幅，在本文就不具体讲述钩子启动程序的实例，只将编写启动程序应注意的事项说明如下：

	- 将NotePadHook项目中Debug\NotePadHook.lib加入到项目设置链接标签中。
	- 将NotePadHook项目中NotePadHook.h文件include到stdafx.h。
	- 首先需要创建一个CNotePadHook类实例，启动钩子时调用类成员StartHook()，卸载钩子时调用类成员StopHook()。
 
##利用窗口子类化(SubClass)拦截消息

前面已提及，每个窗口都有一个在它的窗口类中定义的窗口过程。该窗口过程处理每个发送到窗口的消息。如果想自己编写窗口过程，修改它的行为是没有问题的。但是，如果该窗口过程属于别人，则将没有源代码进行修改。例如，应用程序中的每个按钮，都是由系统提供的BUTTON窗口创建的，它有完全属于自己的窗口过程。如果想改变该窗口的外观，则不能通过改变它的WM_PAINT处理函数来实现，因为它是不可得的。那么，怎样能改变这些按钮的外观，而无需重新编写原来的控件呢？只要用自己的窗口过程的地址，替换窗口对象的初始窗口过程的地址即可。这种技术也是本节讨论的主题 – **窗口子类化技术**。

###窗口子类化原理
应用程序在创建一个新窗口之前要向Windows系统注册这个窗口的类，首先要填写一个WNDCLASS结构，其中的结构参数lpfnWndProc就是该类窗口函数的地址，接着调用RegisterClass()函数向Windows系统申请注册这个窗口类。这时Windows会为其分配一块内存来存放该类的全部信息，这个内存块称为窗口类内存块。

窗口子类化技术实际上就是改变窗口内存块中的有关参数。由于这种修改只涉及到一个窗口的窗口内存块，因此它不会影响到属于同一窗口类的其它窗口的功能和表现。窗口子类化中最常见的是修改窗口内存块中的窗口函数地址（lpfnWndProc），使其指向一个新的窗口函数，从而改变原窗口函数的处理方法，以达到修改其窗口过程的目的。

###窗口子类化的实现
窗口子类化实现的核心是改变窗口过程的地址，可以通过SDK API提供的几个函数来实现。具体步骤如下：

1. 编写子类化窗口过程函数。该函数必须为标准的窗口过程函数格式即： 

		LRESULT CALLBACK SubClassWndProc ( HWND , UINT , WPARAM , LPARAM ) ; 
    	此函数的参数意义与前面讲述的窗口过程函数参数类似。

2. 调用`GetWindowLong ( hWnd , GWL_WNDPROC ) `函数获得原窗口函数的地址并保存起来；其中参数hWnd为待子类化窗口句柄。
3. 调用`SetWIndowLong ( hWnd , GWL_WNDPROC , SubClassWndProc ) `把窗口函数设置成子类化窗口函数，完成窗口子类化。
 
为了减少子类化过程中繁琐的工作，MFC中提供了对子类化的支持，它简化了子类化过程，利用CWnd类SubClassWindows()函数来实现子类化。为了让读者对子类化过程有一个直观的认识，下面将利用MFC实现对一个编辑（Edit）控件的子类化。

1. 创建一个从MFC控件类CEdit派生的新控件类CSubEdit。
2. 添加CSubEdit::PreTranslateMessage(MSG* pMsg)

		BOOL CSubEdit::PreTranslateMessage(MSG* pMsg)
		{
		   if( pMsg->message==WM_KEYDOWN&&pMsg->wParam==VK_RETURN)
		   {
		  //当在Edit控件上按下回车键后…
		 …..
		//限于篇幅处理内容略。
		return TRUE;
		}
		CEdit::PreTranslateMessage(pMsg);
		}

3. 在包含此控件的对话框类头文件中控件变量类型从CEdit改为CSubEdit。
4. 在包含此控件的对话框类文件中对Edit控件进行子类化，代码如下：

		HWND HwND;
		GetDlgItem(IDC_SUB_EDIT,&hWnd);//其中IDC_SUB_EDIT是控件ID。
		m_subEdit.SubclassWindow(hWnd); //m_subEdit为控件变量名。
 
##小结

本文讨论了实现消息拦截的两种方法，其中钩子技术用途广泛，不仅可以实现对同进程内消息的拦截，而且还可以实现对另外进程消息的拦截。而子类化技术主要用于实现对同一进程单元模块中的窗口消息的拦截。程序员可以根据实际应用需求选择其一来实现消息的挡截