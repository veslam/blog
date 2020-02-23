---
title: 从iOS/Android移植Windows需作出的一些修改
date: 2017-02-19 11:57:40
tags: [iOS, Android, Windows, coding, pthread, VisualStudio, glsl]
---
__pthread不认识相关报错__
Windows下需要明确包含库
>\#include < pthread.h>

__实现android提供的函数 clock_gettime(CLOCK_REALTIME, &result);__
http://stackoverflow.com/questions/5404277/porting-clock-gettime-to-windows

__sleep()函数__
>\#include < windows.h> 使用 Sleep()

__Xcode创建的源码文件，有中文字符，VS理解不能__
用NotePad++打开源代码，编码 从UTF-8无BOM格式 -> UTF-8
Configuration Properties -> General -> Character Set => Use Unicode Character Set / Use Multi-Byte Character Set

__dll, lib, .h 文件添加套路__
dll      ——>动态链接库     
include  ——>头文件        C/C++ -> General -> Additional Include Directories
lib      ——>静态链接库     文件目录增加到：Linker -> General -> Additional Library Directories
                          文件名增加到：Linker -> Input -> Additional Dependencies


__Cocos2d-x实现 Resources/ 资源文件 自动拷贝到exe输出目录供调用__
http://www.cocos2d-x.org/wiki/Chapter_2_-_How_to_Add_a_sprite
注意 我改为了 "..\Resources"

__cocos2d win32 glsl 不认识 highp 导致shader无效__
删掉.fsh/.vsh中的 __highp__ 即可运行，未发现问题。