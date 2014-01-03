---
layout: post_m
title:  "实例讲解C++中CopyFileEx函数的简单用法"
date:   2011-02-12 15:38:49
categories: Win32
summary: 
src : 
---
#说明
复制文件可以用 CopyFile API 函数。CopyFileEx 函数的好处是：它提供了回调函数，程序员可以向用户显示复制的过程。

这里我不打算介绍回调函数的详细参数，这个可以看MSDN；我说一些MSDN中没有说明的东西。

1. 多久回调一次？答案是：每传输 64KB 数据回调一次；
2. 回调原因第一次返回：`CALLBACK_STREAM_SWITCH`，以后都返回：`CALLBACK_CHUNK_FINISHED`。第一次表示开始复制，但还没有复制。
3. 回调函数返回 `PROGRESS_CONTINUE`，表示继续复制文件；返回 `ROGRESS_CANCEL`，表示中断复制，这个比较有用。

#示例
上边都是理论，下面我们看一个例子，该例子为 VC++ 6.0 控制台工程。建立一个控制台工程，选择一个空的工程，建立一个 C++ 文件，把下面代码复制进去即可。 


	//
	//    CopyFileEx 回调函数应用


	#define        _WIN32_WINNT    0x0500            // 不加这个不同通过编译
	#include <windows.h>
	#include <stdio.h>

	DWORD CALLBACK CopyProgress(
	    LARGE_INTEGER TotalFileSize,            // total file size, in bytes
	    LARGE_INTEGER TotalBytesTransferred,    // total number of bytes transferred
	    LARGE_INTEGER StreamSize,                // total number of bytes for this stream
	    LARGE_INTEGER StreamBytesTransferred,    // total number of bytes transferred for this stream
	    DWORD dwStreamNumber,                    // the current stream
	    DWORD dwCallbackReason,                    // reason for callback
	    HANDLE hSourceFile,                        // handle to the source file
	    HANDLE hDestinationFile,                // handle to the destination file
	    LPVOID lpData                            // passed by CopyFileEx
	)
	{
	    static int nRecord = 0;
	    nRecord++;
	    printf("回调次数：%d    已传输：%08X:%08X    文件大小：%08X:%08X ",
	        nRecord,
	        TotalBytesTransferred.HighPart,
	        TotalBytesTransferred.LowPart,
	        TotalFileSize.HighPart,
	        TotalFileSize.LowPart);

	    return PROGRESS_CONTINUE;
	}
	int main(int argc, char* argv[])
	{
	    if(argc!=3)
	    {
	        printf("用法：命令 源文件 目标文件");
	        return 0;
	    }

	    if(!CopyFileEx(argv[1],argv[2],(LPPROGRESS_ROUTINE)CopyProgress,NULL,FALSE,COPY_FILE_FAIL_IF_EXISTS))
	    {
	        printf("CopyFileEx() failed.");
	        return 0;
	    }
	    return 0;
	}
#运行结果

	回调次数：1    已传输：00000000:00000000    文件大小：00000000:00F60964
	回调次数：2    已传输：00000000:00010000    文件大小：00000000:00F60964
	回调次数：3    已传输：00000000:00020000    文件大小：00000000:00F60964
	回调次数：4    已传输：00000000:00030000    文件大小：00000000:00F60964
	回调次数：5    已传输：00000000:00040000    文件大小：00000000:00F60964
	回调次数：6    已传输：00000000:00050000    文件大小：00000000:00F60964
	回调次数：7    已传输：00000000:00060000    文件大小：00000000:00F60964
#分析

大家看第一次，已传输为0，从第二次开始，每次传输为 0x10000，为 64KB 字节。