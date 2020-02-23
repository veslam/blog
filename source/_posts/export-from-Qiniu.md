---
title: 博客图片告别七牛移至GitHub
date: 2018-11-07 10:44:13
tags: [Qiniu, GitHub]
---
从建立Hexo主页开始，就按前辈们的推荐使用七牛云作图片存储。平心而论体验算是令人满意，但傻瓜式操作也放纵了我的两个错误行为：1.图片名随机杂乱，2.图片没有保留其他备份。

七牛于今年7月多调整规则，测试域名有效期为3个月，到期后无法使用。而我在10月多收到通知短信才发现图片已无法加载，只道旧图们可能无法寻回，懊悔不已。

好在工单很快得到七牛工程师的回复，按步骤成功将旧图全部下载至本地。
用几小时将图片规范命名，并上传GitHub，这样既有备份，又可直接使用其URL(raw/)作为图片外链。

---
现转帖步骤于此：

> 您好，
> 1.您绑定自定义域名后可以继续使用
> 
> 2.如果您没有域名，可以用下面方法下载
> 有两种方式来获取文件：
> 
> 2.1
> 您需要先新建一个同区域存储空间，会分配一个新的测试域名到新空间。
> 通过qshell batchcopy 到有域名的同区域空间然后再进行qdownload下载操作
> 1）qshell listbucket 原bucket名 list.txt （list出全部文件，https://github.com/qiniu/qshell/blob/master/docs/listbucket.md）
> 2）cat list.txt | awk '{print $1}' >list_final.txt （ 用awk获取list结果的第一列）
> 3）qshell batchcopy 原bucket名 新bucket名 list_final.txt （复制到新bucket的文件和原bucket文件名一致，https://github.com/qiniu/qshell/blob/master/docs/batchcopy.md）
> 4）qshell qdownload newfilelist.txt （newfilelist.txt为下载的配置文档，https://github.com/qiniu/qshell/blob/master/docs/qdownload.md）
> 
> qshell安装包及文档请参考https://developer.qiniu.com/kodo/tools/1302/qshell
> 如果您不熟悉命令行工具的安装使用，也可以结合文档最后提供的视频教程 https://developer.qiniu.com/kodo/tools/1302/qshell#9
> 
> 2.2
> 使用工具qrsctl
> https://developer.qiniu.com/kodo/tools/1300/qrsctl
> qrsctl get

每个步骤的参数设定在后面的链接内都有说明，注意qdownload步骤配置中的"cdn_domain"应设置为新的测试域名。