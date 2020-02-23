---
title: 集成cpr库于Xcode-Cocos2d-x-iOS工程中
date: 2017-05-10 00:15:03
tags: [cpr, Xcode, Cocos2d-x, iOS, C++]
---
### 简介
[cpr (C++ Requests: Curl for People)](https://github.com/whoshuu/cpr)是一个封装了[cURL](https://curl.haxx.se/)的开源库，它把HTTP的几个Request (GET/PUT/POST/DELETE等)封装成了接口，用户需要Request的时候，不用再亲自与cURL打交道繁琐地拼装了。封装程度很高，令人兴奋！
（找到这样一个不错的库，耗费了我两天时间…… 感谢这个列表[A list of open source C++ libraries](http://en.cppreference.com/w/cpp/links/libs)）
>Here's a quick GET request:

``` C
#include <cpr/cpr.h>

int main(int argc, char** argv) {
    auto r = cpr::Get(cpr::Url{"https://api.github.com/repos/whoshuu/cpr/contributors"},
                      cpr::Authentication{"user", "pass"},
                      cpr::Parameters{{"anon", "true"}, {"key", "value"}});
    r.status_code;                  // 200
    r.header["content-type"];       // application/json; charset=utf-8
    r.text;                         // JSON text string
}
```
---
### 集成
然而[ReadMe](https://github.com/whoshuu/cpr/blob/master/README.md)中推荐的，编译[example project](https://github.com/whoshuu/cpr-example)的教程，目标平台是Unix/Win（实测产出了正确运行的Unix executable文件），编译出的libcpr.a不是为iOS运行所用，教程无法套用在Xcode的iOS项目中。

因此我们要把cpr相关的源码、依赖的库加入Xcode项目中编译。

#### 1. 增加相关源代码
工程中建立如下目录结构
- cpr/
    - include/
        - cpr/ ...(1)
        - curl/ ...(2)
        - json.hpp

其中，
__json.hpp__ 拷贝自 cpr-example/opt/json/src/json.hpp
__(1)__ 拷贝自两个来源 cpr-example/opt/cpr/include/cpr/全部文件(.h)
和 cpr-example/opt/cpr/cpr/全部文件(.cpp)
__(2)__ 拷贝自 cocos2d-x-3/external/curl/include/ios/curl/全部源文件
详见文末附录。

Xcode工程文件 Build Settings -> Header Search Paths 增加一项：刚建立的cpr/include目录路径，比如"$(PROJECT_DIR)/cpr/include/"  目的是让头文件能被找到。

#### 2. 增加相关依赖库（.a）
Xcode工程文件 Build Phases -> Link Binary With Libraries 增加：
cocos2d-x-3/external/curl/prebuilt/ios/下的三个.a文件，包括：
>libcrypto.a
libcurl.a
libssl.a

至此，项目编译链接应都通过，可以运行起来。

---
### 开发
目前运行基本样例代码：
``` C++
auto response = cpr::Get(cpr::Url{"https://httpbin.org/get"});
auto json = nlohmann::json::parse(response.text);
cocos2d::log("%s", json.dump(4).c_str());
```

待解决Warning: Libinfo call to mDNSResponder on main thread

运行第一句即报错，response中有错误信息 response.error.message:
__"SSL certificate problem: unable to get local issuer certificate"__
原因可能如[此文](http://blog.kankanan.com/article/4fee590d-ssl-certificate-problem-unable-to-get-local-issuer-certificate.html)所说，iPad上缺少CA证书。

解决方法：关闭验证。通过查看[项目issue](https://github.com/whoshuu/cpr/issues/137)，发现作者已经提供了包装cURL设置是否启用验证的函数，设置cpr::Session即可：
``` C++
session.SetVerifySsl(cpr::VerifySsl(false))
```
所以前述样例代码，要写得展开一些：
``` C++
cpr::Session session;
session.SetVerifySsl(cpr::VerifySsl(false));

auto url = cpr::Url{"https://httpbin.org/get"};
session.SetUrl(url);

auto response = session.Get();
    
// "EXPECT_EQ" requires google test. #include <gtest/gtest.h>
//    EXPECT_EQ(200, response.status_code);
//    EXPECT_EQ(ErrorCode::OK, response.error.code);
    
auto json = nlohmann::json::parse(response.text);
cocos2d::log("%s", json.dump(4).c_str());
```


更多源码可参考cpr项目的[test](https://github.com/whoshuu/cpr/tree/master/test)目录。


---
### 附录：源文件列表checklist：

__(1)__
>api.h
auth.cpp
auth.h
body.h
cookies.cpp
cookies.h
cpr.h
cprtypes.cpp
cprtypes.h
curlholder.h
defines.h
digest.cpp
digest.h
error.cpp
error.h
low_speed.h
max_redirects.h
multipart.cpp
multipart.h
parameters.cpp
parameters.h
payload.cpp
payload.h
proxies.cpp
proxies.h
response.h
session.cpp
session.h
ssl_options.cpp
ssl_options.h
timeout.cpp
timeout.h
util.cpp
util.h

__(2)__
>curl.h
curlbuild-32.h
curlbuild-64.h
curlbuild.h
curlrules.h
curlver.h
easy.h
mprintf.h
multi.h
stdcheaders.h
typecheck-gcc.h
