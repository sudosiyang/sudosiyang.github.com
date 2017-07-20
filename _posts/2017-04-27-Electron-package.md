---
layout: post
title: Electron builder 打包和自动更新
category : 前端技术
tags : 前端 javascript
---

### 打包工具

* [electron-packager](https://www.npmjs.com/package/electron-packager)
* [electron-bulider](https://www.npmjs.com/package/electron-bulider)


用 electron-packager 打包的命令是这样的：
```shell
electron-packager <location of project> <name of project> <platform> <architecture> <electron version> <optional options>
```
以上命令：

将目录切换到项目所在路径，参数 ‘name of project’ 是你的项目名，参数 ‘plateform’ 确定了你要构建哪个平台的应用（Windows、Mac 还是 Linux）,
参数 ‘architecture’ 决定了使用 x86 还是 x64 还是两个架构都用，
决定了使用的 Electron 版本。
第一次打包应用需要比较久的时间，因为所有平台的二进制文件都需要下载，之后打包应用会比较快了。
记得打包参数里写上 overwrite ，不然后面打包时不会重新构建。

electron-builder 就比较折腾了

首先config是个大活，能把文档看懂也需要时间

二进制文件下载外网下载比较不好下，可以改成镜像

`.npmrc`配置为

```
electron_mirror="https://npm.taobao.org/mirrors/electron/"
```

这样就会自动切换到淘宝源，下载electron就比较快了

`build.json`这个是打包配置

```
{
    "directories": {
        "output": "输出目录",
        "app": "目标目录"
    },
    "electronVersion": "electron版本号",
    "mac": {
        "icon": "logo.icns"
    },
    "win": {
        "icon": "logo.ico"
    },
    "publish": {
        "provider": "generic",
        "url": "xxx"
    },
    "productName": "项目名称"
}

```
`provider`这个具体查看文档，我没用aws因为国内比较慢，用的是`generic`自定义的服务器
`url` 自动更新地址


builder可以打包dmg，exe等。这个比较强大

> 但是放到服务器上打包就稍微会麻烦一些，首先你要运行一个winedocker否则无法打包window的包。貌似官方提供一个docker来打包win的exe

packager打包就只能构建app文件夹，还要自己将app文件夹用asar打包





