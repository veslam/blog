---
title: fleeting-weekend
date: 2016-07-24 16:43:48
tags: [diary, Hexo]
---
早上看到老板在群里分享的文章：[16岁用计算器写游戏，55岁临终前启动「Pokemon GO」，42岁就成为任天堂社长的他，人生就是一个大写的程序员！](https://mp.weixin.qq.com/s?plg_nld=1&scene=23&mid=2247484178&plg_uin=1&plg_usr=1&plg_dev=1&sn=cc3d102cc41a770d574387bec0d18a67&plg_nld=1&idx=1&__biz=MzI0NzM5ODQ3Ng%3D%3D&srcid=0719NTLI5xQYAyNkxLPYwjxq&plg_vkey=1&pass_ticket=wNMJLqMr6LbQecqgDG7Cgw%2F9E5y%2BQu4BDb9BABKSwiT3rxl1OClMBGUuDdl5eJ2a#rd&appinstall=0)
原来他就是我一直崇拜的，在红白机时代用汇编语言写游戏的人。看到文章最后，公司总部上空出现彩虹，以及员工对天堂里的岩田聪真挚的告慰，我感动得流泪。文中通篇写的是他多么神，谈笑间樯橹灰飞烟灭，可能有意忽略了他也曾经经历的各种挫折和涅槃，尽管如此，通过他的眼睛能明显感觉到同龄人大多被消磨殆尽的乐观、真诚、热爱。

Hexo继续设置相关：
看到[建议增加social_icons知乎和豆瓣的图标](https://github.com/iissnan/hexo-theme-next/issues/997)，从而了解到[fontAwesome](http://www.cnblogs.com/wangfupeng1988/p/4129500.html)，经测试，现在应该是仍未支持豆瓣、知乎等。
困扰了两天的categories/tags/about页面无内容的问题，今天终于是[解决了](https://www.zhihu.com/question/29017171/answer/112843168#showWechatShareTip)，可气的是主题制作者给出的相同解决方法，出现在了正常显示后的about页面里：
``` bash
### Enable
1. Rename `themes\icarus\_config.yml.example` to `themes\icarus\_config.yml`;
2. Copy `themes\icarus\_config.yml.site.example` to your hexo blog's root directory and rename it to `_config.yml`;
3. Copy `themes\icarus\_source\*` into your hexo blog's directory `source`; # <- This line.
4. Then modify `theme` setting in `_config.yml` to `icarus`.
```

最后，依大家推荐，用七牛作图床。[点我注册😛](https://portal.qiniu.com/signup?code=3llzb2gv67adu) 名字听起来挺专业，其实可以理解为个人存图片的云盘（想存其他文件当然也行），上传后可以得到url用于贴到博客里。（图片如下，大概拍摄于两年前）
![七牛图床成功haha 然而尺寸没法调🙃](https://raw.githubusercontent.com/veslam/blog/master/res/20160724_01_Weekend.jpg)
