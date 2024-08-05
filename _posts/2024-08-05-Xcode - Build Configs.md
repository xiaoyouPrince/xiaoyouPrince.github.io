---
title: Xcode - Build Configs
auther: xiaoyouPrince
date: 2024-08-05 23:54:31 +0800
categories: Xcode build_config
tags: xcode build configs
layout: post
---

# Xcode - Build Configs

> 本文记录了 Xcode 使用 Config 文件自定义 Build Settings 的操作
{: .prompt-info }

## Build Settings 是什么

Build Settings 是编译器在编译阶段的各种配置, 比如编译架构, 目标路径, 语言版本等等等等. 可以简单将配置项分为两类, 系统预置配置项和用户自定义配置项. 

本文主要讲通过使用 `build configuration (xcconfig) file` 方式来配置自定义编译设置, 和为 target 自定义一些文本常量

## build configuration (xcconfig) file

`build configuration (xcconfig) file` 即 xcode build 配置文件, 通过它我们可以在 Xcode 外部通过纯文本的形式编辑编译配置

### 创建

`File -> New -> Configuration Settings File. -> Next -> Create`

这样即创建完成一个 `[SpecifiedFileName].xcconfig file`

### 映射到项目的 build configuration 中

新建的 Xcode 项目, scheme 中可以配置 run 的 configuration, 默认系统会创建 DEBUG / RELEASE 两个.

我们可以降默认的两个编译配置, 映射成我们自己创建的 `[SpecifiedFileName].xcconfig file`, 这样, 对应我们不同的开发环境创建对应的 target.scheme 并 map 到对应的 .xcconfig.file 中.

**具体操作:**

Project -> Info -> Configurations -> 根据不同环境, 配置自己的 xcconfig 文件

### 将 Build Settings 项目拖拽到 xcconfig 文件中

可以打开项目 build setting 和新建的 xcconfig 文件, 拖拽想要编辑的配置项目

## 通过 xcconfig 配置一些常量

比如开发中, 经常区分 pre 环境和 release 环境, 为了方便我们也会创建不同的 scheme 来管理.

例如 pre-config.xcconfig 文件如下

```
BACKSLASH = /
BASE_URL = http:$BACKSLASH/debug.com/api

CHANNEL = develop
```

release-config.xcconfig 文件如下

```
BACKSLASH = /
BASE_URL = http:$BACKSLASH/release.com/api

CHANNEL = develop
```

对此我们可以创建两套编译环境文件, 即 debug 环境有 pre / release 两套配置文件,  release 环境也有 pre / release 两套配置文件. 对此在我们的 scheme 中可以随意配置指定 build setting

### 上述常量的使用

上面的如 `BASE_URL` 本质是被系统设置成了一个编译期的变量. 可以将其写入到 Info.plist 文件中,如在 Info.plist 中新增一个自定义 key, 并设置值为指定变量.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>BASE_URL</key>
	<string>${BASE_URL}</string>
</dict>
</plist>
```

这样我们就可以在项目中通过 Bundle 来读取 info.plist 中的指定 key 的值

```swift
func bundleBaseURL() -> String {
   let mainBundle = Bundle.main
   let BASE_URL = mainBundle.object(forInfoDictionaryKey: "BASE_URL")
   return BASE_URL as! String
}
```

-----
THE END. 


#### 参考

[What is a build setting](https://help.apple.com/xcode/mac/11.4/#/dev382dac089)

[Add a build configration file](https://help.apple.com/xcode/mac/11.4/#/deve97bde215)




