---
title: Cocos2d[3.x]的Label-cc.Label与fnt制作
date: 2018-05-16 12:29:03
tags: [Cocos2d, cc.Label, lua, fnt]
---
备注：本文更新为lua版本，JS版本类名&方法名基本一致。

参考链接：
http://shahdza.blog.51cto.com/2410787/1560612
http://blog.csdn.net/chinahaerbin/article/details/39994261

在Cocos2d[3.x]中，以cc.Label取代[2.x]版本中的cc.LabelTTF, cc.LabelBMFont, cc.LabelAtlas。转而统一通过cc.Label提供的方法创建：cc.Label:createWithSystemFont(), cc.Label:createWithBMFont(), cc.Label:createWithCharMap()。
3种方法创建出的都是cc.Label，但可以视为实际创建出3种子类对象，后续支持的属性设置也不一样。

我目前常用LabelTTF和LabelBMFont，分别对两位的构造方法举例。
- LabelTTF:
``` lua
local labelTTF = cc.Label:createWithSystemFont('', 'Futura', 44) -- 内容，系统字体名，字号
labelTTF:setString('sample text')
labelTTF:setTextColor(cc.c4b(0,255,0,255))
labelTTF:enableShadow(cc.c4b(0,0,0,255), cc.size(0, -2))
```

- LabelBMFont:
``` lua
local labelBMFont = cc.Label:createWithBMFont('xxx.fnt', '') -- fnt路径，内容
```
__不支持设置字号、颜色、阴影等，这些效果都应在fnt制作中完成。__

---

##### 如何制作fnt

- 一套fnt字体包括一个.fnt文件和一个.png文件，与_TexturePacker_导出的.plist+.png类似。
.png存储的是字符图像集合，.fnt记录了每个字符切图在图片里的坐标，显示时的位置参数等。
- 字符集是可以自定义的，只包含数字，只包含字母等等都可以。

制作fnt字体的工具，Windows是[Bitmap Font Generator](http://www.angelcode.com/products/bmfont/)。
据此网址推荐，macOS可用[Glyph Designer](https://71squared.com/glyphdesigner)，确实十分强大易用。
试用版导出会加水印，通常不能忍，所以这是一个[原谅版](http://www.zhinin.com/glyph_designer-mac.html)。

---

##### _Glyph Designer_ 简单教程

- 界面上方三个标签分别是：字体设计页面，效果预览，字符集选择页面。
![](https://github.com/veslam/ImagesForBlog/raw/master/res/20180516_01_Glyph.gif)

- 先选择字符集（根据实际需要显示什么）比如只要数字,/,' '，只要大小写英文 等等。
选完返回设计页面，会看到字符范围已更新。
![](https://github.com/veslam/ImagesForBlog/raw/master/res/20180516_02_Glyph.gif)

- 情况一：美术已给字符切图，直接从finder拖入即可。
![](https://github.com/veslam/ImagesForBlog/raw/master/res/20180516_03_Glyph.gif)

- 预览页面选中 Show Guides显示参考线，通过xAdvance 调整X方向字符间缩进，Y方向位移。
![](https://github.com/veslam/ImagesForBlog/raw/master/res/20180516_04_Glyph.jpg)

- 导出。产出为 fnt + png。
![](https://github.com/veslam/ImagesForBlog/raw/master/res/20180516_05_Glyph.jpg)

- 情况二：（比较少见）需要自己根据已知系统字体直接制作。调整各参数即可。
![](https://github.com/veslam/ImagesForBlog/raw/master/res/20180516_06_Glyph.gif)

