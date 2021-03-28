---
title: 我的MacOS自我救赎之路（指电脑坏了重新买）
date: 2020-05-20 08:48:14
tags: 
  - MacOS
categories:
  - memo
---



## 装机指南

### 常规软件

#### App Store

```sh
1.QQ
2.Wechat
3.Netease Music
4.Magnet
5.Youku
...
```

#### otherTools

```sh
1.iTerm2
2.uTools
3.VSCode
4.VMWare
5.Jetbrains Toolbox
6.XunLei Thunder
7.Baidu Disk
8.WPS
9.Charles
10.Docker
11.KeyCastr (录屏时显示键盘)
12.v2ray
```

#### Exists

```
1.ForkLift
2.Chrome
3.JDK8
4.Karabiner-Elements(key-bind-change)
5.Microsoft Remote Desktop
6.RDM
7.ShadowsocksX NG
8.Sougou Input
9.SublimeText
10.Typora
```

#### Safari 扩展

```
1.AdBlock (App Store)
2.Save to Pocket (App Store)
3.Toast for Safari (App Store)
4.Vimari (Github)
```

#### IDEA 插件

```
1.Alibaba Java Coding Guidelines
2.ignore
3.Translation
4.Lombok
5.VisualVM Launcher
6.GenerateAllSetter
7.Git Commit Template
```



### 开发工具

#### 命令行一套

```sh
# zsh + oh my zsh + 高亮 + vim 高亮
curl -L https://raw.githubusercontent.com/gudqs7/mycode/master/zsh/one_zsh.sh | sh

# Git 别名
curl -L https://raw.githubusercontent.com/gudqs7/mycode/master/git/git-alias.sh | sh

# git代理
git config --global http.https://github.com.proxy http://127.0.0.1:1087
git config --global https.https://github.com.proxy https://127.0.0.1:1087
```

#### ssh 代理

`vi ~/.ssh/config`

```sh
# 保持心跳
Host *
    ServerAliveInterval 60

# git
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    ProxyCommand /usr/bin/nc -x 127.0.0.1:1086 %h %p
    IdentityFile ~/.ssh/id_rsa
    
Host bitbucket.org
    HostName bitbucket.org
    ProxyCommand /usr/bin/nc -x 127.0.0.1:1086 %h %p
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_bucket

Host e.coding.net
    HostName e.coding.net
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_bucket


```



#### Homebrew

##### 安装 HomeBrew 及设置源

```sh
#安装home brew
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

#设置加速源(若有科学上网设置了 git 代理后, 不建议设置加速源)
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

brew update

echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```

##### 添加 taps

```sh
#以太坊
brew tap ethereum/ethereum
#cask
brew tap homebrew/cask
brew tap homebrew/cask-versions
#core
brew tap homebrew/core
#services
brew tap homebrew/services
#php
brew tap josegonzalez/php
```

##### 添加常用小工具

```sh
brew install telnet
brew install nmap
brew install vim
brew install wget
brew install git-flow
#cat 增强版
brew install bat
#man 简化版
brew install tldr
#find 增强版
brew install fd
#top 增强版
brew install htop
brew install tree
brew install go python node pyenv
brew install mysql redis postgresql
#k8s 相关
brew install docker-machine docker-machine-driver-hyperkit kubernetes-cli
brew install hadoop
brew install rabbitmq
brew install nexus
brew install siege
brew install ffmpeg
brew install ethereum solidity
```

##### brew cask

```sh
# finder 快速预览增强
brew cask install provisionql qlprettypatch quicklook-json betterzip qlcolorcode qlstephen quicklookapk qlimagesize qlvideo quicklookase qlmarkdown quicklook-csv

brew cask install minikube
brew cask install keycastr

```

#### Nodejs

##### 全局 npm 包

```sh
npm install -g asar composer-cli pm2
#wx unpacker
npm install -g cnpm n nrm npm
npm install -g css-tree cssbeautify escodegen esprima js-beautify uglify-es vm2
npm install -g electron-builder electron-packager electron-prebuilt
npm install -g ganache-cli remix-ide truffle
npm install -g vue-cli yo
```



### 目录参考

#### code/remote

```sh
.
├── gitee
│   ├── Mapper
│   └── jeesite4
├── github
│   ├── gudqs7
│   │   ├── JVMStudy
│   │   ├── architecture
│   │   ├── grid-helper
│   │   ├── gudqs7.github.io
│   │   ├── idea-keymap
│   │   ├── memo
│   │   ├── mycode
│   │   ├── php-study
│   │   ├── poi-tl
│   │   ├── utools-plugin
│   │   └── wxappUnpacker
│   ├── gudqs7-team
│   │   ├── an-jie-api
│   │   ├── miniprogram-demo
│   │   └── noworry-mini
│   ├── java
│   │   ├── SpringBootBucket
│   │   └── netty
│   └── other
│       ├── QQPlugin-MacOS
│       └── WeChatPlugin-MacOS
└── other
```



### 所有 APP 参考

```sh
#ls /Applications
AdBlock.app                              HBuilderX.app                            SunloginControl.app
AirServer.app                            JD-GUI.app                               TeamViewer.app
Alfred 4.app                             JetBrains Toolbox.app                    Tencent Lemon.app
Antivirus One.app                        Karabiner-Elements.app                   Thunder.app
AppCleaner.app                           Karabiner-EventViewer.app                Toast.app
Aria2GUI.app                             KeyCastr.app                             Typora.app
Axure RP 9.app                           Keynote.app                              Utilities
BaiduNetdisk_mac.app                     Luyten.app                               VMware Fusion.app
BetterZip.app                            Magnet.app                               Vimari.app
Blackmagic Disk Speed Test.app           Microsoft Excel.app                      Visual Studio Code.app
CCtalk.app                               Microsoft Remote Desktop Beta.app        WeChat.app
Charles.app                              Microsoft Word.app                       Weiyun.app
CheatSheet.app                           NemuPlayer.app                           Wireshark.app
Clarity Wallpaper Mac.app                NeteaseMusic.app                         XMind.app
CleanerOne.app                           Numbers.app                              Xcode.app
Clover Configurator.app                  Outline Manager.app                      apps
DingTalk.app                             Outline.app                              balenaEtcher.app
Docker.app                               Pages.app                                iMovie.app
DrUnarchiver.app                         Postman.app                              iPic.app
Eclipse.app                              QQ.app                                   iTerm.app
FileZilla.app                            QQLive.app                               mat.app
Firefox.app                              RDM.app                                  sPlayerFree.app
ForkLift.app                             RedisDM.app                              uTools.app
Ganache.app                              Safari.app                               wechatwebdevtools.app
GarageBand.app                           Save to Pocket.app                       优酷.app
Geekbench 4.app                          ShadowsocksX-NG.app                      微信支付商户平台证书工具.app
Google Chrome.app                        Sketch.app
```



### 镜像下载及U盘制作

镜像下载地址:

```
网址: imac.hk
```

U盘制作工具

```
etcher
https://www.balena.io/etcher/
```

