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
> 3. View 协议当作类型使用
> 4. @StateObject 和 @EnvironmentObject
> 5. @ViewBuilder
> 6. View.mask 和 View.overlay
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

## View 协议当作类型使用

View 是个协议, 无法直接当作类型使用, 可以使用 `any View` 代替, 或者返回具体的 View 类型

```swift
func getView() -> any View {
   return Text("")
}

func getView() -> some View {
   return Text("")
}
```

## @StateObject 和 @EnvironmentObject

@StateObject 和 @EnvironmentObject 都是用于数据双向绑定对象的注解, 用于修饰 ObservableObject 的对象

@StateObject 用于声明 View 的成员变量, 需要初始化赋值.

@EnvironmentObject 用于声明环境变量, 通过 View.environmentObject() 赋给视图树 RootView 赋值, 之后整个视图树内的 View 可直接声明使用(由系统统一赋值).


```swift
class DataModel: ObservableObject {
    @Published var name: String = "点我试试"
}

struct BannerView: View {
    // @StateObject var dataModel: DataModel
    @EnvironmentObject var dataModel: DataModel
    
    var body: some View {
        Text(dataModel.name)
            .onTapGesture {
                dataModel.name = "试试就试试"
            }
    }
}

// BannerView(dataModel: DataModel())
BannerView().environmentObject(DataModel())
```

> @EnvironmentObject 需要当前 View 添加到视图树中才会被系统赋值<br>
> 同样只有 View 被加入到视图树中, View.onAppear 函数才会被调用<br>
> 若 View 展示时需要加载网络数据, 更新 dataModel<=> View, 则必须先将 View 添加到视图树
> 
> 这就意味着: 当子视图使用环境变量的时候, 若未被安装到视图树, 则环境变量因未赋值而 Crash
{: .prompt-tip }


## @ViewBuilder

@ViewBuilder 是一个注解, 如下代码会将函数体中所有 View 封装成一个整体 View 返回, 
如果不用 @ViewBuilder, 则需要显示写明 return 某个具体 View

```swift
@ViewBuilder
func getView() -> some View {
   Text("")
   Image("")
}
```

## View.mask 和 View.overlay



-----
THE END. 






