---
title: 使用Electron打包HTML成EXE
date: 2020-05-20 09:02:46
tags: 
  - electron
categories: 
  - electron
---

### Windows 下 (推荐) 

#### 下载 nodejs 和 git

> 下载 nodejs 32 位 msi 安装包 (https://npm.taobao.org/mirrors/node/v12.13.1/node-v12.13.1-x86.msi)
>
> 下载 git 的安装包 (https://git-scm.com/download/win) (可能贼慢, 自行解决)

#### 安装 nodejs 及 cnpm

> 首先将 node-v12.13.1-x86.msi 双击安装, 一路 next 就行了
>
> 然后将 git 也装上, 也是一路 next, 不要犹豫

安装淘宝cnpm

```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

#### 下载 hello-world 项目

>  找到一个空白文件夹, 准备放置项目, 右键 选择 Git bash here, 然后命令行中输入: 

```shell
git clone https://github.com/gudqs7/my-electron-hello-world.git
```

下载好后, 不要关闭窗口, 继续下面操作

#### 安装依赖

在命令行中, 进入文件夹, 输入命令安装依赖

```shell
cd my-electron-hello-world
cnpm i
```

#### 试运行下 App

```shell
cnpm run dev
```

> 看到一个宽1200 高 850 的窗口里面写着一句 `Hello world` 没有! 那就成了.

#### 打包 exe

```shell
#一步到位, 生成 xxx setup.exe 安装包. (推荐)
cnpm run dist
#仅生成文件夹, 内含 xxx.exe 可启动, 很多资源文件, 需要自己二次打包
cnpm run pack
```

> 打包成功后, 可在dist 文件夹下找到 exe 名称 Setup 1.0.1.exe, 复制拿走即可





### MacOS 下

#### 安装 brew 及 设置 brew 源镜像为清华源

自行谷歌, 不建议百度

#### 安装 nodejs 和 cnpm (淘宝镜像 npm)

打开终端(建议使用 iTerm2), 输入命令: 

```shell
brew install nodejs
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

#### 下载 hello-world 项目

>  找到一个空白文件夹, 准备放置项目, 然后进命令行并 cd 到这个目录下, 输入命令: 

```shell
git clone https://github.com/gudqs7/my-electron-hello-world.git
```

下载好后, 不要关闭窗口, 继续下面操作

#### 安装依赖

在命令行中, 进入文件夹, 输入命令安装依赖

```shell
cd my-electron-hello-world
cnpm i
```

#### 试运行下 App

```shell
cnpm run dev
```

> 看到一个宽1200 高 850 的窗口里面写着一句 `Hello world` 没有! 那就成了.

#### 打包成 Mac App

```shell
#生成 dmg 文件, 方便分享, 下载
cnpm run dist
#生成可直接运行的 App, 但实际上是个文件夹, 不方便给他人下载
cnpm run pack
```

> 打包命令执行完毕, 检查 dist 目录即可

#### 打包成 windows exe

```shell
cnpm run win32
```

> 这个命令我自己都下载半天, 实在是久啊. 而且由于要使用 wine 啥的进行打包, 似乎失败率很高啊...... 建议试试 yarn(官方建议)
>
> 个人推荐直接用 windows 操作系统搞一下就行了



### 注意事项

```shell
platform = win32  # 这个字段的值, 使用 --win 命令可指定, 默认与电脑环境一致
arch=ia32         # 这个字段的值, 使用--ia32 或 --x64 指定, 建议 ia32 省的 32 位系统不兼容 64 位的程序
```

> 另外多说一句, 若使用 32 位 windows 系统, cnpm i 下载依赖时就会下载 electron 的一个 sdk(用于生成 exe 的核心文件), 这个 sdk 是区分平台和cpu架构的. 由于用命令行指定平台和 cpu 架构不仅麻烦, 而且他使用的下载地址, 网速偏慢(极慢), 而 cnpm i 下载时则相对较快(感谢马云), 所以推荐选择适当的操作系统来打包, 会比较快.

- 另外, 尝试使用 `electron .` 这个命令在 `windows` 上是跑不来的, 但将其加入到 package.json 中的 script 然后使用 `cnpm run xxx` 就没问题了.

- 送大家一个网址, 在线 ico 转换, 支持256x256 (https://www.aconvert.com/cn/icon/png-to-ico/)