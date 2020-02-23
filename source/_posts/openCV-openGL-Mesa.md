---
title: openCV-openGL-Mesa
date: 2016-12-29 11:05:22
tags: [OpenCV, OpenGL, Mesa]
---
切记！
VisualStudio包含Mesainclude文件夹！

相关问题解决

[__未声明M_PI error C2065: 'M_PI' : undeclared identifier__](http://www.cnblogs.com/JackieWu/p/4341596.html)
>声明使用M_PI的默认方法为：
>
>在stdafx.h头文件中声明.
>
``` C
#define _USE_MATH_DEFINES 
#include <math.h>
```
（记得#include "stdafx.h"哦）

[__Error: “fopen': This function or variable may be unsafe.”__](http://stackoverflow.com/a/21873153/2918210)

>__2.040 What OpenGL implementations come with source code?__ [[Link]](https://www.opengl.org/archives/resources/faq/technical/gettingstarted.htm)
>The [Mesa library](http://www.mesa3d.org/) is an OpenGL look-alike. It has an identical interface to OpenGL. The only reason it can't be called "OpenGL" is because its creator hasn't purchased a license from the OpenGL ARB.
The [OpenGL Sample Implementation](http://oss.sgi.com/) is also available.

TOadd tables providing solutions

>Mesa 3D是一个在MIT许可证下开放源代码的三维计算机图形库，以开源形式实现了OpenGL的应用程序接口。OpenGL的高效实现一般依赖于显示设备厂商提供的硬件，而Mesa 3D是一个纯基于软件的图形应用程序接口。由于许可证的原因，它只声称是一个“类似”于OpenGL的应用程序接口。由于Mesa 3D的api是和opengl 相同，具体的opengl版本浏览Mesa 3D官方网站，我们可以这么认为它就是opengl的软件模拟gpu光栅处理器的一个实现。我们知道如果要实现一个opengl，其本身是一个设备器，不能实现窗体的透明，如果我想要实现窗体透明，又想要有3D的应用，可以试试它。
[[Link]](http://download.csdn.net/detail/lixeb/4876142)

><GL/glut.h>：GLUT（OpenGL实用工具包）所使用的函数和常量声明。这个库的功能大致与GLAUX类似，目前许多OpenGL教程使用这个库来编写演示程序。一些编译系统可能不直接提供这个库（例如VC系列），需要单独下载安装。这个头文件自动包含了<GL/gl.h>和<GL/glu.h>，编程时不必再次包含它们。 
[[Link]](http://www.cnblogs.com/laizhd/archive/2011/06/18/2084133.html)