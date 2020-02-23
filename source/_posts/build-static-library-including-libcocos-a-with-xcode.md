---
title: Build Static Library 包含libcocos.a with Xcode
date: 2016-09-15 21:01:54
tags: [static library, Cocos2d-x, Xcode, coding]
---
趟过N个坑，参考M个网页，千辛万苦Build成功。把遇到的issues记在这里供借鉴。

## 〇. 用Cocos2d-x源码编译libcocos.a
### 参考网页[[Tutorial] build cocos2d-x as a static library in Mac Xcode](http://fstoke.me/blog/?p=4067)
(这一步并不是必须的，如果你能下载到他人已编译好的.a文件也行。)

##### 1. 建立Cocos环境
Cocos源码可从官网得到，以terminal进入源码根目录，运行命令
```
python setup.py
```

##### 2. 建立新项目

>首先，要開一個cocos2d的專案。到你下載並解壓好的cocos2d-x根目錄(例: /Users/YourName/Library/cocos2d-x-3.6)執行以下指令
```
cocos new HelloWorld -p com.CompanyName.HelloWorld -l cpp
```
>接著打開Xcode project檔。
```
open HelloWorld/proj.ios_mac/HelloWorld.xcodeproj
```

##### 3. 修改Build Settings

>選擇cocos2d_libs.xcodeproj，再選擇Target: libcocos2d iOS，修改其Build Settings。
應該預設值就是下面寫的值了，如果不是，請改為下面的值。
```
Architectures => Standard architectures
Build Active Architecture Only => No
```
>另外在Valid Architectures 那欄追加兩個值: i386 和 x86_64。
這是為了讓build出來的Library也可以在模擬器上跑。
修改後的結果應該有五個值: arm64, armv7, armv7s, i386, x86_64
完整結果如圖片所示:
![https://lh3.googleusercontent.com/-wnOpXEUQAXs/VY4uFgdXWvI/AAAAAAAAJFU/GMvuVionjUI/s1291/cocos2d-x-project-setting.png](https://github.com/veslam/ImagesForBlog/raw/master/res/20160915_01_Build.png)

##### 4. 使用Rakefile來建立Static Library

> 
```
cd HelloWorld/cocos2d/build
vi Rakefile
```
>內容輸入以下程式碼:
```
PROJECT_PATH = "./cocos2d_libs.xcodeproj"
TARGET_NAME="'libcocos2d iOS'"
OUTPUT_DEBUG="tmp/iphonesimulator"
OUTPUT_RELEASE="tmp/iphoneos"
OUTPUT_LIB="./lib"
directory OUTPUT_LIB

desc "using lib command to build a static library"
task "lib" do
    sh "xcodebuild -project #{PROJECT_PATH} -configuration Release -sdk iphonesimulator8.3 -target #{TARGET_NAME} -arch i386 -arch x86_64 TARGET_BUILD_DIR=#{OUTPUT_DEBUG} BUILT_PRODUCTS_DIR=#{OUTPUT_DEBUG} clean build"

    sh "xcodebuild -project #{PROJECT_PATH} -configuration Release -sdk iphoneos8.3 -target #{TARGET_NAME} -arch armv7 -arch armv7s -arch arm64 TARGET_BUILD_DIR=#{OUTPUT_RELEASE} BUILT_PRODUCTS_DIR=#{OUTPUT_RELEASE} clean build"
end

desc "using lipo command to link static libraries for each device to one combined library file"
task "lipo" => OUTPUT_LIB do

    Dir.glob("#{OUTPUT_RELEASE}/*"){|path|
        p path
        file = File.basename(path)

        sh "lipo '#{OUTPUT_DEBUG}/#{file}' '#{OUTPUT_RELEASE}/#{file}' -create -output '#{OUTPUT_LIB}/#{file}'"
    }
end
```

注：iphonesimulator&iphoneos中的版本号可能需要更新，否则有错误：
>SDK "iphonesimulator8.3" cannot be located

解决办法：参考[利用xcodebuild命令确定SDK版本](http://www.it165.net/os/html/201208/3102.html)运行命令：
```
xcodebuild -showsdks
```
查知两者版本号，相应修改Rakefile即可。

>開始製作static library
輸入以下command。
```
rake lib
```
>整個過程大概需要10~20分鐘，要看你的機器的執行速度。如果不到一分鐘就停了，那可能是有出現錯誤。像我是遇到找不到 iphonesimulator8.2 和 iphoneos8.2 這兩個資料夾，要改成你的執行環境正確的值。

>完成build static library後，你會在tmp資料裡看到兩個libcocos2d iOS.a檔，一個是給實機device跑的，另一個是給模擬器跑的。因此我們還要再做最後一步，將這兩個.a檔打包成為一個merged的.a檔。執行下面的指令。
```
rake lipo
```
>這大概只需要幾秒鐘就能跑完，跑完之後你會看到一個lib資料。裡面有一個libcocos2d iOS.a檔，檔案大小約 117.9 MB。這就是最終我們要的static library檔了。

** BUILD SUCCEEDED ** 开心😄
![得到的libcocos.a就可以用啦！https://lh3.googleusercontent.com/-nGnHzsYT5YM/VY4uFL-wp7I/AAAAAAAAJFI/xkhOGuct6r0/s575/cocos2d-x-add-static-library.png](https://github.com/veslam/ImagesForBlog/raw/master/res/20160915_02_Build.png)

## 一. 在目标Xcode工程中加入libcocos.a
这里的目标工程就是指你用来制作Static Library的工程啦。

##### 1. Header Search Paths
头文件方面，依旧使用Cocos源码目录下的.h文件们。
添加如下目录，__切记不要recursive__，否则引起混乱。
```
".../cocos2d-x-3.x", （cocos根目录，请酌情修改。）
".../cocos2d-x-3.x/cocos",
".../cocos2d-x-3.x/cocos/base",
".../cocos2d-x-3.x/cocos/physics",
".../cocos2d-x-3.x/cocos/math",
".../cocos2d-x-3.x/cocos/2d",
".../cocos2d-x-3.x/cocos/ui",
".../cocos2d-x-3.x/cocos/network",
".../cocos2d-x-3.x/cocos/audio/include",
".../cocos2d-x-3.x/cocos/editor-support",
".../cocos2d-x-3.x/extensions",
".../cocos2d-x-3.x/external",
".../cocos2d-x-3.x/external/chipmunk/include/chipmunk",
```
##### 2. Library Search Paths
增加 libcocos.a 所在路径。

***
至此，幸运的话应该能编译通过了。然而我比较脸黑，还有各种小问题的骚扰，记录如下。

## 二. 其它部分Errors的解决

##### - 找不到gl.h
仔细看报错位置，发现是编译mac平台相关头文件时出现。因此猜测是工程文件设置不到位，导致Xcode尝试为mac平台编译。
解决：Build Settings > Apple LLVM 7.1 - Preprocessing > Preprocessor Macros > + following
```
USE_FILE32API
CC_TARGET_OS_IPHONE
COCOS2D_DEBUG=1 (Debug加)
CC_ENABLE_CHIPMUNK_INTEGRATION=1
```

##### - _MacTypes.h: Reference to 'Point' is ambiguous_
这个问题事实很简单：Cocos的Point类名与iOS冲突。然而解决费了我千辛万苦。最终找到标准答案[ERROR: Reference to “Point” is ambiguous in MacTypes.h](http://discuss.cocos2d-x.org/t/error-reference-to-point-is-ambiguous-in-mactypes-h/11853/17?u=veslam)
>In short:
1. Don't let USING_NS_CC; be declared in any of your header files.
2. Don't include Foundation or UIKit in header. Move it into cpp.
3. If you need to use a type from it, hide the implementation into a separate class.

简言之：不要在头文件中使用 USING_NS_CC（.cpp等可以） ，也不要引 _Foundation_ 或 _UIKit_，那我要在头文件中提到Cocos的变量怎么办呢？乖乖加上命名空间_cocos2d::_

> #### Why is "Point" ambiguous in MacTypes.h?
There are two types of __Point__ types, one defined in Cocos2d and other by iOS. The problem rises because the compiler detects references to __Point__ type without specifying which one.
The C++ way is to use a "namespace", like __cocos2d::Point__. 
However, since it's cumbersome to write the prefix each time, there is a convention to declare "I'm going to omit the __cocos2d::__ prefix in this scope. Just regard every __Point__ here as __cocos2d::Point__.". That's the __using namespace cocos2d__ line, or even more convenient, __USING_NS_CC__. This is causing the problem.
If there is any file that declared using namespace cocos2d in a header file, then it gets mixed up during the long include chain. Hence, "you shouldn't use __USING_NS_CC__ in header files".

##### - _ld: file is universal (3 slices) but does not contain a(n) armv7s slice: /file/location for architecture armv7s clang: error: linker command failed with exit code 1 (use -v to see invocation)_
[File is universal (three slices), but it does not contain a(n) ARMv7-s slice error for static libraries on iOS, anyway to bypass?](http://stackoverflow.com/questions/12402092/file-is-universal-three-slices-but-it-does-not-contain-an-armv7-s-slice-err/12402966#12402966)
>Alternatively, you can set the flag for your __debug__ configuration's __"Build Active Architecture Only"__ to __Yes__. Leave the __release__ configuration's __"Build Active Architecture Only"__ to __No__, just so you'll get a reminder before releasing that you ought to upgrade any third-party libraries you're using.

##### - 编译出来的Static Library放到工程中使用，报错：Undefined symbols for architecture arm xxx
参考两篇文章[ios build时，Undefined symbols for architecture xxx问题的总结](http://www.cnblogs.com/piaojin/p/5591656.html)｜[Undefined symbols for architecture arm64解决方案](http://blog.csdn.net/zuoyou1314/article/details/46638073)
简而言之，苹果设备从老到新依次使用这四个指令集（_armv6、armv7、armv7s、arm64_），若想支持更多型号的苹果设备，就要支持更多指令集，编译出来的包体积也就越大。
编译出来的.a的工程叫builder，使用.a的工程叫user，那么在支持的指令集方面，builder需要包含user，提供的才能用。那么久分别设置builder和user工程__Build Settings > Architectures__相关，满足上述条件就行啦。

具体来说，影响Architectures的设置有二：
1. _Architectures_与_Valid Architectures_的交集。前者基本必选_Standard Architectures_不用在意，后者自己决定选好，_Debug_和_Release_没必要不一样。
2. _Build Active Architectures Only_，大意是指只选择当前连接设备的指令集，显然不要这样。设置方法前面说过，_Release_为_No_，_Debug_可为_Yes_。

##### - "_SecCertificateCreateWithData", referenced from:
增加Security.framework

##### - "_GCControllerDidDisconnectNotification", referenced from:
增加MediaPlayer.framework, GameController.framework

##### - "_iconv_open", referenced from:的解决方案
添加动态库 libiconv.dylib（然而我没找到，加libiconv.tbd也可解决。）

##### - "_CTFramesetterCreateWithAttributedString", referenced from:
增加CoreText.framework

***
就酱🤔
