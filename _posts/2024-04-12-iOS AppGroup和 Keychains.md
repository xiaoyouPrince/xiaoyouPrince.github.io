---
title: iOS AppGroup和 Keychains
auther: xiaoyouPrince
date: 2024-04-12 01:21:52 +0800
categories: iOS 零碎
tags: appgroup keychain
layout: post
---

# SwiftUI 编程 Tips

> 本文记录了使用 SwiftUI 编程时需要注意的一些细节. 
> 1. View.onAppear 
{: .prompt-info }

## 前言

SwiftUI 是一个声明式 UI 框架, 它基于 Combine 内置数据双向绑定机制, 在 UI 构建中具有非常大的优势. 但它也存在一些功能短板, 以至于一些自定义程度较高的 UI 组件还是使用 UIKit 更为合适. 

我的看法: 
App 主框架使用 UIKit (众所周知: 国内 App 启动时有很多操作, 各种生命周期也有很多操作)
App 内部的业务 UI 使用 Swift UI 搭建

## View.onAppear

> View.onAppear 类似 UIKit 中 viewDidLoad / viewWillAppear 合体, 需要手动处理仅执行一次的操作. 

View 的生命周期函数 onAppear 类似于 UIKit 中 viewWillAppear. 通常我们会处理一些 view 被展示到屏幕前的事件. 但不同于 UIKit 的是 SwiftUI 中没有提供类似 viewDidLoad 的函数. 对于仅需要处理一次的操作(如读取内存数据)每次都会执行, 并覆盖用户的编辑值

具体原因如下:
SwiftUI 采用 Combine 的数据双向绑定, 其内部会记录很多 @State. 当某个 @State 发生改变, 会触发 UI 重绘制, 会重新执行 View.onAppear, 如果其中包含读取默认值的代码, 则会重新读取默认值, 覆盖当前修改过的 @State.


-----
THE END. 






