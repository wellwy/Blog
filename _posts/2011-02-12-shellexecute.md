---
layout: post_m
title:  "深入浅出ShellExecute"
date:  2011-02-12 18:07:12
categories: win32
description:  
src :  
---

##Q:如何打开一个应用程序？

	ShellExecute(this->m_hWnd,"open","calc.exe","","", SW_SHOW );

或

	ShellExecute(this->m_hWnd,"open","notepad.exe", "c:\\MyLog.log","",SW_SHOW );

正如您所看到的，我并没有传递程序的完整路径。

##Q: 如何打开一个同系统程序相关连的文档？

	ShellExecute(this->m_hWnd,"open", "c:\\abc.txt","","",SW_SHOW );

##Q: 如何打开一个网页？

	ShellExecute(this->m_hWnd,"open", "http://www.google.com","","", SW_SHOW );

##Q: 如何激活相关程序，发送EMAIL？

	ShellExecute(this->m_hWnd,"open", "mailto:nishinapp@yahoo.com","","", SW_SHOW );

##Q: 如何用系统打印机打印文档？

	ShellExecute(this->m_hWnd,"print", "c:\\abc.txt","","", SW_HIDE);

##Q: 如何用系统查找功能来查找指定文件？

	ShellExecute(m_hWnd,"find","d:\\nish", NULL,NULL,SW_SHOW);
##Q: 如何启动一个程序，直到它运行结束？

	SHELLEXECUTEINFO ShExecInfo = {0}; ShExecInfo.cbSize = sizeof(SHELLEXECUTEINFO); ShExecInfo.fMask = SEE_MASK_NOCLOSEPROCESS; ShExecInfo.hwnd = NULL; ShExecInfo.lpVerb = NULL; ShExecInfo.lpFile = "c:\\MyProgram.exe"; ShExecInfo.lpParameters = ""; ShExecInfo.lpDirectory = NULL; ShExecInfo.nShow = SW_SHOW; ShExecInfo.hInstApp = NULL; ShellExecuteEx(&ShExecInfo); WaitForSingleObject(ShExecInfo.hProcess,INFINITE);

或：

	PROCESS_INFORMATION ProcessInfo; STARTUPINFO StartupInfo; //This is an [in] parameter ZeroMemory(&StartupInfo, sizeof(StartupInfo)); StartupInfo.cb = sizeof StartupInfo ; //Only compulsory field if(CreateProcess("c:\\winnt\\notepad.exe", NULL, NULL,NULL,FALSE,0,NULL, NULL,&StartupInfo,&ProcessInfo)) { WaitForSingleObject(ProcessInfo.hProcess,INFINITE); CloseHandle(ProcessInfo.hThread); CloseHandle(ProcessInfo.hProcess); } else { MessageBox("The process could not be started..."); }

##Q: 如何显示文件或文件夹的属性？
	
	SHELLEXECUTEINFO ShExecInfo ={0}; ShExecInfo.cbSize = sizeof(SHELLEXECUTEINFO); ShExecInfo.fMask = SEE_MASK_INVOKEIDLIST ; ShExecInfo.hwnd = NULL; ShExecInfo.lpVerb = "properties"; ShExecInfo.lpFile = "c:\\"; //can be a file as well ShExecInfo.lpParameters = ""; ShExecInfo.lpDirectory = NULL; ShExecInfo.nShow = SW_SHOW; ShExecInfo.hInstApp = NULL; ShellExecuteEx(&ShExecInfo);