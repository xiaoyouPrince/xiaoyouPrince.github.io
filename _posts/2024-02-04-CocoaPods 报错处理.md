---
title: CocoaPods 报错处理
auther: xiaoyouPrince
date: 2024-02-04 18:58:15 +0800
categories: CocoaPods
tags: cocoapods pod
layout: post
---

# CocoaPods 报错处理


> 本文记录了一个 cocoapods 报错的处理, 由于我经常创建 Demo, 频繁处理此问题,遂记录一下
> - CocoaPods 1.14.3
> - Xcode Version 15.0.1 (15A507)
{: .prompt-info }

pod install 成功之后

project run fail:

```
报错: 执行沙盒脚本报错
Sandbox: rsync.samba(99672) deny(1) file-write-create /Users/quxiaoyou/Library/Developer
/Xcode/DerivedData/pod_test-fbafrddymulpqcbiucjvbjtqaami/Build/Products/Debug-
iphonesimulator/pod_test.app/Frameworks/XYNav.framework/_CodeSignature
```

解决:
检索: `ENABLE_USER_SCRIPT_SANDBOXING`

![](/media/CocoaPods 报错处理/1.jpg)

-----
THE END.

### 参考
https://www.youtube.com/watch?v=MoR-vopM1HY 


