---
layout: post_m
title:  "_vscprintf, _vscwprintf"
date:   2011-02-12 15:39:41
categories: Win32
summary: 
src : 
---

#_vscprintf, _vscwprintf
Returns the number of characters in the string referenced by a list of arguments.

	int _vscprintf( const char *format, va_list argptr ); int _vscwprintf( const wchar_t *format, va_list argptr );
#Parameters

*format*

Format-control string.

*argptr*

Pointer to list of arguments.

For more information, see [Format Specifications][1].

#Return Value

`_vscprintf` returns the number of characters that would be generated if the string pointed to by the list of arguments was printed or sent to a file or buffer using the specified formatting codes. The value returned does not include the terminating null character. `_vscwprintf` performs the same function for wide characters.

#Remarks

Each *argument* (if any) is converted according to the corresponding format specification in *format*. The format consists of ordinary characters and has the same form and function as the *format argument* for [printf][2].
**Security Note**   Ensure that format is not a user-defined string. For more information, see [Avoiding Buffer Overruns][3].
**Generic-Text Routine Mappings**


TCHAR.H | routine	\_UNICODE & \_MBCS not defined | \_MBCS defined_UNICODE defined
:------|:------|:------
\_vsctprintf | \_vscprintf | \_vscprintf`

#Requirements

Routine	 | Required header	 | Compatibility
:------|:------|:------
_vscprintf	 | <stdio.h>	 | ANSI, Win 98, Win Me, Win NT, Win 2000, Win XP
_vscwprintf	 | <stdio.h> or <wchar.h> | 	ANSI, Win 98, Win Me

For additional compatibility information, see [Compatibility][4] in the Introduction.

**Libraries**
All versions of the [C run-time libraries][5].

#Example
See the example for [vsprintf][6].


	string CPaiPaiHelper::Format(LPCTSTR fmt, ...)
	{ 
		string strResult="";
		if (NULL != fmt)
		{
			va_list marker = NULL;            
			va_start(marker, fmt);                            //初始化变量参数 
			size_t nLength = _vscprintf(fmt, marker) + 1;    //获取格式化字符串长度
			std::vector<char> vBuffer(nLength, '\0');        //创建用于存储格式化字符串的字符数组
			int nWritten = _vsnprintf_s(&vBuffer[0], vBuffer.size(), nLength, fmt, marker);
			if (nWritten>0)
			{
				strResult = &vBuffer[0];
			}            
			va_end(marker);                                    //重置变量参数
		}
		return strResult; 
	}

[1]:[http://msdn.microsoft.com/zh-cn/library/56e442dc(v=VS.71).aspx]
[2]:[http://msdn.microsoft.com/zh-cn/library/wc7014hz(v=VS.71).aspx]
[3]:[http://msdn.microsoft.com/zh-cn/library/ms717795(v=VS.71).aspx]
[4]:[http://msdn.microsoft.com/zh-cn/library/sk54f3f5(v=VS.71).aspx]
[5]:[http://msdn.microsoft.com/zh-cn/library/abx4dbyh(v=VS.71).aspx]
[6]:[http://msdn.microsoft.com/zh-cn/library/28d5ce15(v=VS.71).aspx]