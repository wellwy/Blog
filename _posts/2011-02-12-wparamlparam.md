---
layout: post_m
title:  "WPARAM与LPARAM的区别"
date:   2011-02-12 15:51:29
categories: Win32
description: 
src : http://www.cppblog.com/expter/archive/2008/12/24/70213.html
---

#WPARAM与LPARAM的区别

具体是这么说

> 在Win 3.x中，WPARAM是16位的，而LPARAM是32位的，两者有明显的区别。因为地址通常是32位的，所以LPARAM 被用来传递地址，这个习惯在Win32 API中仍然能够看到。在Win32 API中，WPARAM和LPARAM都是32位，所以没有什么本质的区 别。Windows的消息必须参考帮助文件才能知道具体的含义。如果是你定义的消息，愿意怎么使这两个参数都行。但是习惯上，我们愿意使用LPARAM传 递地址，而WPARAM传递其他参数。

看一个例子就明白了：  

程序代码

	/*在对话框中取出数据，并向其他窗口发送消息和数据，将数据指针作为一个参数发送*/
	void CTestDlg2::OnCommBtn()
	{
	     char szOut[30];
	      GetDlgItemText(IDC_OUT,szOut,30);
	      m_pParent->SendMessage(WM_DLG_NOTIFY,(WPARAM)szOut);
	}

	ON_MESSAGE(WM_DLG_NOTIFY,OnDlgNotifyMsg)


	void CMy53_s1View::OnDraw(CDC* pDC)
	{
	      CMy53_s1Doc* pDoc = GetDocument();
	      ASSERT_VALID(pDoc);
	     // TODO: add draw code for native data here
	      pDC->TextOut(0,0,"Display String");
	      pDC->TextOut(0,20,m_szOut);
	}

	LONG CMy53_s1View::OnDlgNotifyMsg(WPARAM wP,LPARAM lP)
	{
	      m_szOut=(char*)wP;
	      Invalidate();
	     return 0;
	}

一个字符串的地址通过WPARAM来标识，再通过Windows消息发送出去；之后在消息处理函数中WPARAM接受到的参数就是该地址，然后就可以对该地址进行操作了～～～


这是Windows消息机制中经常用到的两个data type，呵呵。