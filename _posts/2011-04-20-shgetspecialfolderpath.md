---
layout: post_m
title:  "SHGetSpecialFolderPath的用法"
date:   2011-04-20 09:28:26
categories: win32
description:  
src : 
---

##SHGetSpecialFolderPath

###作用：
获取特定文件夹路径
###原型：

	BOOL SHGetSpecialFolderPath(
	         HWND hwndOwner,
	         LPTSTR lpszPath,
	         int nFolder,
	         BOOL fCreate
	);
###示例：

获得自启动文件夹的路径

    TCHAR filePath[MAX_PATH];
    ::SHGetSpecialFolderPath(NULL, filePath, CSIDL_STARTUP, FALSE);

以下是`nFolder`值的对应情况

获取值的机器为多普达838

	CSIDL_STARTMENU —— \Windows\“开始”菜单
	CSIDL_STARTUP —— \Windows\StartUp
	CSIDL_WINDOWS —— \Windows
	CSIDL_RECENT ——
	CSIDL_PROGRAMS —— \Windows\“开始”菜单\程序
	CSIDL_PROGRAM_FILES —— \Program Files
	CSIDL_PERSONAL —— \My Documents
	CSIDL_MYVIDEO ——
	CSIDL_MYPICTURES —— \My Documents\我的图片
	CSIDL_MYMUSIC —— \My Documents\我的音乐
	CSIDL_FONTS —— \Windows\Fonts
	CSIDL_FAVORITES —— \Windows\Favorites
	CSIDL_DESKTOPDIRECTORY ——
	CSIDL_DESKTOP —— \My Documents
	CSIDL_APPDATA —— \Application Data