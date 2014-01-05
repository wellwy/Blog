---
layout: post_m
title:  "打开文件通用对话框"
date:   2011-02-12 16:40:51
categories: Win32
summary: 
src : http://www.cnblogs.com/fromchaos/archive/2009/11/29/1613289.html
---

#GetOpenFileName 

打开文件通用对话框

	BOOL GetOpenFileName(      
	    LPOPENFILENAME lpofn );
使用该函数需要包含 commdlg.h 头文件

	#include <Commdlg.h>
	打开文件通用对话框
	BOOL  OpenFileDialog(HWND hWnd)
	{
		OPENFILENAME ofn;       // common dialog box structure
		TCHAR szFile[MAX_PATH];       // buffer for file name
		// Initialize OPENFILENAME
		ZeroMemory(&ofn, sizeof(ofn));
		ofn.lStructSize = sizeof(ofn);
		ofn.hwndOwner = hWnd;
		ofn.lpstrFile = szFile;
		//
		// Set lpstrFile[0] to '\0' so that GetOpenFileName does not 
		// use the contents of szFile to initialize itself.
		//
		ofn.lpstrFile[0] = '\0';
		ofn.nMaxFile = sizeof(szFile);
		ofn.lpstrFilter = TEXT("Image files (*.jpg;*.bmp)\0*.jpg;*.bmp\0\0");
		ofn.nFilterIndex = 1;
		ofn.lpstrFileTitle = NULL;
		ofn.nMaxFileTitle = 0;
		ofn.lpstrInitialDir = NULL;
		ofn.Flags = OFN_PATHMUSTEXIST | OFN_FILEMUSTEXIST; 
		return GetOpenFileName((LPOPENFILENAME)&ofn);
	}