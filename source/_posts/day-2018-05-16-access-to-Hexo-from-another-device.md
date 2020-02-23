---
title: 开始于iMac@work编辑发布博客
date: 2018-05-16 11:05:58
tags: [Hexo, Google Drive, MarkdownEditing, freeCodeCamp, codepen]
---
做了两件事，一是把博客项目文件目录上传[GoogleDrive](https://drive.google.com/)同步，通过[BackupAndSync](https://www.google.com/drive/download/backup-and-sync/)应用；
二是复制项目到在公司的iMac上并建立Hexo环境。以后就可以在这台设备上更新博客啦。

再次记录一下步骤：
+ 安装[_Homerbrew_](https://brew.sh/)
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
+ 通过 _brew_ 安装 _node_
```
brew install node
```
    可能需要以下命令初始化_npm_的package.json
    ```
    npm install --yes
    ```
+ 通过 _npm_ 安装 [_Hexo_](https://www.npmjs.com/package/hexo)
在博客项目文件目录中运行
```
npm install -g hexo-cli -O && npm install -O
```
    遇到不支持的node_module，卸载重装应即可。

最终，
```
hexo config
```
看看域名、URL都符合自己的配置就对了。

---

##### 最近两个感兴趣的网址
- [freecodecamp](https://www.freecodecamp.org/) 在线任务型教程
目前在看这个[后端](https://www.freecodecamp.org/map-aside#collapseBack-End-Development-Certification)部分，[AWS Cloud9](https://console.aws.amazon.com/cloud9)远程终端很先进的感觉。
- [codepen.io](codepen.io) 有git功能的前端源码项目集
其中 [Musical Chord Progression Arpeggiator](https://codepen.io/jakealbaugh/pen/qNrZyw)是一个很有趣的参数化作曲工具，枚举了乐曲的调式、音高等不同属性。




