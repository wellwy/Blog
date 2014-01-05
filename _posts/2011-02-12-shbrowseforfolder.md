---
layout: post_m
title:  "SHBrowseForFolder函数"
date:   2011-02-12 16:47:01
categories: Win32
summary: 文件目录选择函数SHBrowseForFolder()及其回调函数
src : 
---

#SHBrowseForFolder函数

一般的OpenDialog，得到的是文件夹名称，如果要想实现下面的效果，得到选择的路径，这个时候`SHBrowseForFolder`就派上用场了。

## 示例     
下面的例子中返回路径，如果没有选，返回"",选择了路径，则返回选择的路径。

	char *GetPath(HWND   hWnd,char   *pBuffer)  
	{  
	    BROWSEINFO   bf;  
	    LPITEMIDLIST   lpitem;  
	    memset(&bf,0,sizeof   BROWSEINFO);  
	    bf.hwndOwner=hWnd;  
	    bf.lpszTitle="选择路径";  
	    bf.ulFlags=BIF_RETURNONLYFSDIRS;     //属性你可自己选择  
	    lpitem=SHBrowseForFolder(&bf);  
	    if(lpitem==NULL)     //如果没有选择路径则返回   0  
	        return   "";  
	   
	    //如果选择了路径则复制路径,返回路径长度  
	   
	    SHGetPathFromIDList(lpitem,pBuffer);  
	    return    pBuffer;
	}

##解释

下面我们来解释一下这个函数用到的一些值的含义。
   
### BROWSEINFO
    
它是一个结构， 原型是

    typedef struct _browseinfo {
	    HWND hwndOwner;          // 弹出的dialog的父窗体的句柄
	    LPCITEMIDLIST pidlRoot; // 指向一个ITEMIDLIST的指针，我们会在后边介绍ITEMIDLIST结构，可空
	    LPSTR pszDisplayName;    // 指向一个buffer,这个buffer用来存放用户选中的目录，buffer的size为MAX_PATH  
	    LPCSTR lpszTitle;        //指向一个非空的string，用来显示树目录之上的指示信息
	    UINT ulFlags;            // 指出了显示的文件夹的类型
	    BFFCALLBACK lpfn;        //简单的记为NULL    
	    LPARAM lParam;           // 当lpfn不为空时，把dialogbox的值传给回调函数lpfn
	    int iImage;              // 系统的图标list的索引，当用户选中目录的时候，得到这个索引
	} BROWSEINFO, *PBROWSEINFO, *LPBROWSEINFO;

ulFlags的可能取值为：

	BIF_BROWSEFORCOMPUTER 	//只返回"我的电脑",当选中"我的电脑"之外的目录时,OK键为灰色
	BIF_BROWSEFORPRINTER 	//只返回"打印机",当选中"打印机"之外的目录时,OK键为灰色
	BIF_DONTGOBELOWDOMAIN	//不包括"网上邻居"
	BIF_RETURNFSANCESTORS	//只返回"我的文件",当选中"我的文件"之外的目录时,OK键为灰色
	BIF_RETURNONLYFSDIRS	//同上
	BIF_STATUSTEXT 			//Includes a status area in the dialog box. The callback can set the status text by sending messages to the dialog box.
### ITEMIDLIST
   
是一个结构，指明了默认浏览的根文件夹的位置，可以为空，那样的话，默认为桌面文件夹的文件目录.

原型为

	typedef struct _ITEMIDLIST
	{  
	    SHITEMID mkid; // list of item identifers
	} ITEMIDLIST, * LPITEMIDLIST;
	typedef const ITEMIDLIST * LPCITEMIDLIST;

## SHGetPathFromIDList函数
   
原型是
   
	WINSHELLAPI BOOL WINAPI SHGetPathFromIDList(
	    LPCITEMIDLIST pidl,
	    LPSTR pszPath
	);

反正记住配套使用就行了，哈哈~~

如果是在bcb环境中使用，那么如果提示不能识别BROWSEINFO,则需加入头文件`＃i nclude <ShellAPI.h>`,然后在对应的.cpp的include之前`#define   NO_WIN32_LEAN_AND_MEAN`

##消息

### SHBrowsForFoler() -> 回调函数
目录浏览对话框可能会像回调函数发送3种消息：

+ **BFFM_INITIALIZED** :通知对话框已经初始化结束。

	回调函数响应此消息时通常是做初始选择

+ **BFFM_SELCHANGED**:目录浏览对话框当前选择项发生变化时调用此消息。

	回调函数响应此消息时通常是显示所选项的相关信息

+ **BFFM_VALIDATEFAILED**:表示用户按确认按钮时却发现浏览对话框的编辑框内输入了一个非法名称

	回调函数响应此消息时通常是提示客户选择项非法，并确定是否继续显示该对话框

### 回调函数 -> SHBrowsForFoler()

回调函数可以发送如下几个消息给目录浏览对话框，从而改变目录浏览对话框的面目

+ **BFFM_SETSELECTION**:改变当前选择项目
+ **BFFM_ENABLEOK**: 改变“确认”按钮的状态
+ **BFFM_SETSTATUSTEXT**:改变目录浏览对话框中状态行消息，当然前提是目录浏览对话框中有状态行