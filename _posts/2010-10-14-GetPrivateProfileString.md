---
layout: post_m
title:  "有关GetPrivateProfileString的使用方法"
date:  2010-10-14 17:22
categories: win32
description:  
src :  
---


函数返回值为string的长度(long型),而从ini文件获得的字符串则保留在目的缓冲器中 

	DWORD GetPrivateProfileString( 
	LPCTSTR lpAppName, //配置文件的section名 
	LPCTSTR lpKeyName, //配置文件的key名 
	LPCTSTR lpDefault,  
	LPTSTR lpReturnedString,  
	DWORD nSize,  
	LPCTSTR lpFileName  
	);  

其中各参数的意义:  

- 前二个参数与 WritePrivateProfileString中的意义一样. 
- **lpDefault** : 如果INI文件中没有前两个参数指定的字段名或键名,则将此值赋给变量.
- **lpReturnedString** : 接收INI文件中的值的CString对象,即目的缓存器. 
- **nSize** : 目的缓存器的大小. 
- **lpFileName** : 是完整的INI文件名. 


下面是一个常见的出错原因： 

>GetPrivateProfileString怎么总是读不出来 


*.INI内容 

	[NETWORK] 
	ServerIP=100.100.100.53 

程序: 

	main() 
	{ 
	char ip[16]; 
	DWORD num=0; 
	num=GetPrivateProfileString("NETWORK","ServerIP,"", 
	       ip,sizeof(ip), 
	   "Server.ini"); 
	cout<<num<<endl<<ip<<endl; 
	} 


>`num=GetPrivateProfileString("NETWORK","ServerIP,"", ip,sizeof(ip),"Server.ini");` 
>
>Server.ini这个文件放在哪里的?要放在与应用程序相同的目录下应该用".\\server.ini" 
>
>你看看是不是没有找到这个INI文件 

>VC中调试时，server.ini放在工程目录中；程序单独运行时，则需要放在跟exe同一个目录中。 
>
>因为从VC里启动程序，VC将程序的工作目录初始化为工程目录，而不是debug或release目录本身。