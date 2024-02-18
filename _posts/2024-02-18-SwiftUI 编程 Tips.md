---
title: SwiftUI 编程 Tips
auther: xiaoyouPrince
date: 2024-02-18 14:30:05 +0800
categories: 零碎知识 编程Tips
tags: swiftUI UIKit
layout: post
---

# SwiftUI 编程 Tips

> 本文记录了使用 SwiftUI 编程时需要注意的一些细节. 
> 1. View.onAppear 
> 2. 使用 GeometryReader 获取基于父视图的布局空间
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

## 使用 GeometryReader 获取基于父视图的布局空间

GeometryReader 是一个特殊 View. 它像函数一样以其父视图的布局空间来包装指定子视图. 

其主要目的是对子视图布局提供运行时的正确布局空间数据, 如获取父视图的 width/height.

通常, 我们只直接声明式布局 UI 即可, 比如 ZStack / HStack / VStack 等来布局, SwiftUI 会自动进行布局,并合理利用空间. 但是, SwiftUI 默认是根据子 View 来填充,撑开父容器. 这就有一个问题: 当子视图尺寸明显大于我们指定父视图尺寸的时候, 如果父视图被设置了固定尺寸, 此刻子视图会因为尺寸过大而超出父视图可是范围, 其超出部分并非以左上角原点为锚点对齐,从而造成布局异常.

GeometryReader 就是为了解决这问题, 当父容器固定尺寸, 子视图尺寸过大, 它会读取运行时父容器布局空间, 并以左上角为锚点对齐(真实锚点应该是平台有关, 本文以 iOS 为例.)


-----
THE END. 






