---
title: IDEs for Android on Mac 踩坑实录
date: 2017-1-15 11:05:22
tags: [Android, IDE, coding, eclipse, Android Studio]
---

### eclipse
自行安装eclipse，除了配置SDK／NDK，还需配置ADT [eclipse中离线安装ADT插件详细教程](http://blog.csdn.net/dr_neo/article/details/46941859)

志明同学指导："有快捷方式图标的，都是android目录外的。"

__⚠️ 注意！不要修改工程的ndk设置__
而要直接设置 eclipse -> 偏好 C/C++ -> Build -> Environment + 'NDK_ROOT'

__发现了以元素 'd:skin' 开头的无效内容。此处不应含有子元素。__
sdk目录\tools\lib下的devices.xml替换掉出错的devices.xml，然后重新启动eclipse。

---
### Android Studio (Recommended)

用eclipse而不熟悉AndroidStudio的人，建议先读资料：[名词概念辨析](http://ask.android-studio.org/?/article/20)
>Eclipse 的 __Project__ 等同于Android Studio的 __Module__ 。
Eclipse 的 __workspace__ 等同于Android Studio的 __Project__ 。

以下两条分割线之间的内容，均摘录自文章[在 Android Studio 2.2 中愉快地使用 C/C++](http://blog.csdn.net/wl9739/article/details/52607010)

---
使用 Android studio，你可以将 C 和 C++ 代码构建成 native library（即 .so 文件），然后打包到你的 APK 中。你的 Java 代码可以通过 Java Native Interface（JNI）调用 native library 库中的方法。

Android Studio 默认使用 CMake 编译原生库。由于已经有大量的代码使用了 ndk-build 来编译 native code，所以 Android Studio 同样也支持 ndk build。如果你想导入一个 ndk-build 库到你的 Android Studio 项目中，请参阅后文的 关联本地库与 Gradle 部分。然而，如果你创建了一个新的 native 库工程，你应该使用 CMake。

#### 下载 NDK 和构建工具
要编译和调试本地代码（native code），你需要下面的组件：
+ _The Android Native Development Kit (NDK)_: 让你能在 Android 上面使用 C 和 C++ 代码的工具集。
+ _CMake_: 外部构建工具。如果你准备只使用 ndk-build 的话，可以不使用它。
+ _LLDB_: Android Studio 上面调试本地代码的工具。

你可以使用 SDK Manager 来安装上述组件
1. 打开一个项目，从菜单栏中选择 Tools > Android > SDK Manager。
2. 点击 SDK Tools 选项卡。
3. 勾选 LLDB，CMake 和 NDK。
4. 点击 Apply，然后点击 OK。
5. 当安装完成后，点击 Finish，然后点击 OK。

#### 将 C/C++ 代码添加到现有的项目中
如果你想将 native code 添加到一个现有的项目中，请按照下面的步骤操作：
1. 创建新的 native source 文件，并将其添加到你的 Android Studio 项目中。如果你已经有了 native code，也可以跳过这一步。
2. 创建一个 CMake 构建脚本。如果你已经有了一个 CMakeLists.txt 构建脚本，或者你想使用 ndk-build 然后有一个 Android.mk 构建脚本，也可以跳过这一步。
3. 将你的 native library 与 Gradle 关联起来。Gradle 使用构建脚本将源码导入到你的 Android Studio 项目中，并且将你的 native library （也就是 .so 文件）打包到 APK 中。
一旦你配置好了项目，你就可以在 Java 代码中，使用 JNI 框架开调用原生函数（native functions）。只需要点击 Run 按钮，就可以编译运行你的 APP 了。

#### 创建新的 native source 文件
请按照下面的方法来创建一个 cpp/ 目录和源文件（native source files）：
1. 打开IDE左边的 Project 面板，选择 Project 视图。
2. 找到你项目的 module > src 目录，右键点击 main 目录，选择 New > Directory。
3. 输入目录的名字（比如 cpp），然后点击 OK。
4. 右键点击刚才创建好的目录，选择 New > C/C++ Source File。
5. 输入文件名，比如 native-lib。
6. 在 Type 菜单下拉选项中，选择源文件的扩展后缀名，比如 .cpp。
7. 如果你也想创建一个头文件，点击 Create an associated header 选项框。
8. 点击 OK。

---
### Android Studio - import cocos2d project (FAILED)
cocos2d自动创建的工程，proj.android本意是给eclipse使用的。好在Android Studio导入时能判断它是eclipse工程，并提示：将会拷贝新建自用的proj.android而不会覆盖。一路下一步就可以啦。

遇到第一个问题：
__Error: Your project contains C++ files but it is not using a supported native build system.__
__Consider using CMake or ndk-build integration with the stable Android Gradle plugin:__
    https://developer.android.com/studio/projects/add-native-code.html
__or use the experimental plugin:__
    http://tools.android.com/tech-docs/new-build-system/gradle-experimental.

---
### Android Studio - create cocos2d project (FAILED)
follow instruction [Cocos command-line tool
](http://cocos2d-x.org/docs/editors_and_tools/cocosCLTool/)(这篇官方教程教授的是用命令行实现从建立工程到编译apk的整个过程)

1. "python setup.py" in cocos2d root folder
2. "cocos new _<project name_> -p com.MyCompany.MyGame -l cpp -d _<location_>"
3. "cocos compile -s _<path to your project_> -p android __--android-studio__ [-m debug] --ap android-25" ［注意⚠️］
+ 命令里含有"--android-studio"，这是专门针对Android Studio进行编译。如果不运行这一步，原生C++代码无法运行。[[link](http://m.blog.csdn.net/article/details?id=49026035)]
+ "--ap android-version"是Android必需的 奇怪的是，我这里在version=25时成功，10,13,16等较低版本都报错：
__./platform/CCFileUtils.cpp:274: error: undefined reference to 'atof'__

*上面的第2,3步可能遇到的问题及解决如下，若涉及修改 ~/.bash_profile 的操作，之后都要执行 source ~/.bash_profile 来使设置生效。*
__NDK_ROOT / ANDROID_SDK_ROOT / ANT_ROOT 无效__
~/.bash_profile 增加如下行
```
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
```
```
export NDK_ROOT=/Users/.../android-sdk-macosx/ndk-bundle
export ANDROID_SDK_ROOT=/Users/.../android-sdk-macosx
export ANT_ROOT=/Users/.../apache-ant-1.8.1/bin
```
［注意］__ANT_ROOT__要指向/bin子目录，否则会报错：__.../apache-ant-1.8.1/ant: No such file or directory__

__Error: JAVA_HOME is not defined correctly.__
~/.bash_profile 增加
```
export JAVA_HOME=$(/usr/libexec/java_home)
export PATH=$JAVA_HOME/jre/bin:$PATH
```

terminal出现类似下面语句，即表示成功。
>BUILD SUCCESSFUL
Total time: 15 seconds
Move apk to _<path to your project_>/bin/debug/android
Build succeed.

__'Open existing Android Studio project'时卡在 building 'proj.android-studio' gradle project info__
打开该目录下gradle/wrapper/gradle-wrapper.properties，最后一行
```
distributionUrl=https\://services.gradle.org/distributions/gradle-版本号-all.zip
```
原因：正在下载相应版本的gradle，然而被墙卡住。
+ 最好方法：通过VPN让它自己下下来。
+ 方法二：手动下载相应版本的gradle-版本号-all.zip，不解压置于/Users/username/.gradle/wrapper/dists/相应版本号/一串乱码/ 目录下。
+ 方法三（没办法的办法，并不总能奏效）：将版本号替换为本地已安装的gradle版本号，具体可从AndroidStudio设置中查到，或从/Users/username/.gradle/wrapper/dists/中查到。

最终，terminal中出现绿色"Build succeed."字样，表示工程成功生成，用Android Studio import "/proj.android-studio" 即可。可以在真机运行HelloWorld，然而cpp部分并不在工程中，猜测可能是提前编译为.so文件，供Java调用，这仍不是我们想要的。

---
#### Cocos2d-x相关问题解决
__import com.loopj.android.http.AsyncHttpClient__ [回答贴链接](http://discuss.cocos2d-x.org/t/cocos2d-x-3-9-eclipse-problem-solution/25060)
解决方法：add jar 'cocos2d-x\cocos\platform\android\java\libs\android-async-http-1.4.8.jar'
__相关知识 [add jars和add external jars有什么区别？](http://blog.csdn.net/haqer0825/article/details/8183264)__ 
>android中导入第三方jar包的正确方式
1，右键工程-->Build path-->java build path,
2，选择libraries在右边的按钮中点击“Add Library”    
3，选择“User library”,点击“下一步”     
4，点击“User librarys”按钮在出现的界面中点击“New..”按钮,在弹出的界面中随便起一个名字，点击“确定”   
5，点击“Add jars”按钮选择第三方jar包，点击“确定”完成操作。这样的话该jar包会被一起打包到apk中，问题也就解决了

---
#### 其他小问题解决记录
__How to delete a module in Android Studio__
File -> Project Structure, select a Module, click "minus" button. [[link](http://stackoverflow.com/a/24592192/2918210)]

__如何修改minSdkVersion__
在build.gradle文件中. [[link](http://stackoverflow.com/a/20449862/2918210)]

__"unknowm skin name nexus_5"__
未知原因导致的AVD问题，直接在AVD manager里删除重建模拟器就解决了。
