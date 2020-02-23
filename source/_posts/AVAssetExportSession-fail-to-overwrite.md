---
title: AVAssetExportSession-fail-to-overwrite
date: 2016-08-04 13:23:12
tags: [coding, iOS, AVFoundation]
---
先上结论：
+ iOS应用在卸载之前，沙盒路径是不变的。
+ __AVFoundation__通过__AVAssetExportSession__写文件到沙盒中时，无法覆盖已有文件，会失败。

---

下面是心路历程。
从昨天下午，App单机运行时有如下表现：
1. 有概率会在[处理视频/存储视频/播放视频]这个过程中的某一步闪退，具体在哪一步当时还没有确定。
2. 一旦闪退，以后重启App，必定在相似位置闪退。（猜测是在同一步，后来确认的确是。）
3. Xcode连机运行，虽然也有概率在相似位置闪退，但下次运行就没问题了啊。🤔

首先想明白的是第3点。Xcode连机的每次运行，其实相当于覆盖安装一次App，所以和单机跑是有区别的（其实回过头来看这也是个线索，闪退一定和两者的区别有关）。

之后没有头绪，希望能尽量确定闪退的代码位置。Xcode连机时似乎不能不安装只重启App，单机运行不能打断点，log输出也看不到，那就只好把log显示到屏幕上，看显示到那一条时闪退。写到一个TextField里显示出来（只能显示最新一条）。
经过多次在代码里挪动输出的位置，基本确定问题出在存储视频这一步。

那么，要存的文件肯定是好的，存储失败最大的可能就是，文件名已存在且覆盖失败？
沙盒tmp文件夹路径格式是：
>/var/mobile/Containers/Data/Application/440E4599-DAFF-4B57-AD79-759248479F1B/tmp/

我的文件名格式是A-B.mp4，A来自截取沙盒路径里的16进制部分，B是从0开始递增的整数，所以文件名是：
>440E4599-DAFF-4B57-AD79-759248479F1B-0.mp4

之所以设计得这么简单，是因为我觉得既然每次运行，沙盒路径都不一样，那多次运行App的视频名称自然不可能相同，因此只需要加一个序号后缀，区分本次运行生成的文件们就行啦。（教训，还是不要偷懒，最好用NSDate等能保证自己唯一性的东西。）
难道，其实沙盒路径是固定的？而连机调试时之所以每次运行显示沙盒路径都不一样，是因为重新安装的结果？？？经测试，果真，沙盒路径不会变。……

之后看到自己写了却一点印象也没有的 assert(exporter.error == nil); 一定是闪退在这里啦，也是醉了。
so，闪退流程：沙盒路径不变 -> 视频文件重名 -> 存储失败 -> assert() -> 闪退

最后一个问题，__AVAssetExportSession__真的无法覆盖已存在的同名文件吗？是的，[官方讲解](https://developer.apple.com/library/ios/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/01_UsingAssets.html)：
>The export will fail if you try to overwrite an existing file, or write a file outside of the application’s sandbox. It may also fail if:
+ There is an incoming phone call
+ Your application is in the background and another application starts playback
In these situations, you should typically inform the user that the export failed, then allow the user to restart the export.