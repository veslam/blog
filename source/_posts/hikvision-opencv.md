---
title: hikvision-opencv
date: 2016-11-27 21:06:04
tags: [HIKVISION, OpenCV]
---
获取海康威视网络摄像头图像，并转换为OpenCV图像格式。
参考：[捕获海康威视IPCamera图像，转成OpenCV可以处理的图像（二）](http://blog.csdn.net/wanghuiqi2008/article/details/31410509) [第二链接](http://www.tuicool.com/articles/biIZja)
亲测可用，文中所说转化为YUV实际上是指转化为YCrCb。

---
海康威视 MFC例程 VC6 用VS2013打开遇到问题和解决

__Building an MFC project for a non-Unicode character set is deprecated. You must change the project property to Unicode or download an additional library.__

❌ project properties -> general -> project default -> character set. 改为 Unicode
✅ http://stackoverflow.com/a/24772665/2918210
you need to install MFC MBCS DLL Add-on As mentioned in your error. (下载后成功解决！)
https://msdn.microsoft.com/zh-cn/library/dn251007(v=vs.120).aspx
https://www.microsoft.com/en-us/download/confirmation.aspx?id=40770

__cannot convert parameter 1 from 'const char [6]' to 'const wchar_t *__

http://stackoverflow.com/a/18162514/2918210
the parameter has to be wrapped inside a _T or TEXT macro:
如 strIP.Format("%d.%d.%d.%d",add4,add3,add2,add1); 变为 strIP.Format(_T("%d.%d.%d.%d"),add4,add3,add2,add1);

__Visual C++中UpdateData()函数的功能是什么？afxwin.h__
>UpdateData，顾名思义，是用来刷新数据的。UpdateData(TRUE) -- 刷新控件的值到对应的变量   UpdateData(FALSE) -- 拷贝变量值到控件显示例如，窗口中用 DDX_Text(pDX, IDC_EDIT1, m_usercode); 将IDC_EDIT1编辑框控件与m_usercode变量做了关联，如果修改m_usercode之后要想对应控件显示更改，则需要调用UpdateData(FALSE);反之在IDC_EDIT1的oneditchanged()中需要加入UdateData(TRUE);   简单的说，如果Updatedata(TRUE) == 将控件的值赋值给成员变量;Updatedata(FALSE) == 将成员变量的值赋值给控件;
