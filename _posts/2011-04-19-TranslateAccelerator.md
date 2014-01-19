---
layout: post_m
title:  "TranslateAccelerator"
date:   2011-04-19 11:34:50
categories: win32
description:  
src :  
---


##相关介绍

**函数功能**

翻译加速键表。该函数处理菜单命令中的加速键。该函数将一个`WM_KEYDOWN或WM_SYSKEYDOWN`消息翻译成一个`WM_COMMAND或WM_SYSCOMMAND`消息（如果在给定的加速键表中有该键的入口），然后将`WM_COMMAND`或`WM_SYSCOMMAND`消息直接送到相应的窗口处理过程。
 
`TranslateAccelerator`直到窗口过程处理完消息后才返回。
 
**函数原型**

	int TranslateAccelerator（HWND hWnd,HACCEL hAccTable，LPMSG IpMsg）;
**参数**
 
- `hWnd`:窗口句柄，该窗口的消息将被翻译。
- `hAccTable`:加速键表句柄。加速键表必须由LoadAccelerators函数调用装入或由`CreateAccd_eratorTable`函数调用创建。
- `LpMsg`:MSG结构指针，MSG结构中包含了从使用GetMessage或PeekMessage函数调用线程消息队列中得到的消息内容。
 
**返回值**

若函数调用成功，则返回非零值；若函数调用失败，则返回值为零。若要获得更多的错误信息，可调用GetLastError函数。

**备注**

为了将该函数发送的消息与菜单或控制发送的消息区别开来，使`WM_COMMAND`或`WM_SYSCOMMAND`消息的`wParam`参数的高位字值为1。用于从窗口菜单中选择菜单项的加速键组合被翻译成`WM_SYSCOMMAND`消息：所有其他的加速键组合被翻译成`WM_COMMAND`。若`TransLateAccelerator`返回非零值且消息已被翻译，应用程序就不能调用`TranslateMessage`函数对消息再做处理。每个加速键不一定都对应于菜单命令。若加速键命令对应于菜单项，则`WM_INITMEMU`和`WM_INITMENUPOPUP`消息将被发送到应用程序，就好像用户正试图显示该菜单。然而，如下的任一条件成立时，这些消息将不被发送：窗口被禁止，菜单项被禁止。
 
##加强键组合

加速键组合无相应的窗口菜单项且窗口己被最小化。鼠标抓取有效。有关鼠标抓取消息，参看SetCapture函数。若指定的窗口为活动窗口且窗口无键盘焦点（当窗口最小化时一般是这种情况）， `TranslateAccelerator` 翻译`WM_SYSDEYUP`和`WM_SYSKEYDOWN`消息而不是`WM_KEYUP`和`WM_KEYDOWN`消息。
 
当按下相应于某菜单项的加速键，而包含该菜单的窗口又已被最小化时， `TranslateAccelerator` 不发送`WM_COMMAND`消息。但是，若按下与窗口菜单或某单项的任一项均不对应的加速键时， `TranslateAccelerator` 将发送一`WM_COMMAND`消息，即使窗口己被最小化。
 
**Windows CE**：所有的加速键消息被翻译成`WM_COMMAND`消息；Windows CE不支持`WM_SYSCOMMAND`消息。
 
**速查**：Windows NT：3.1 及以上版本；Windows：95及以上版本：Windows CE：1.0及以上版本；

**头文件**：windows.h;库文件：user32.lib; Unicode：在Windows NT实现为Unicode和ANSI两种版本。