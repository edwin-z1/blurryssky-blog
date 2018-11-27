---
title: PullingHeader
date: 2018-03-21 18:33:35
tags:
	- iOS开发 
	- custom UI
---

## 闲聊一下

😅年会的时候公司给每个人发了一部iphoneX，看来苹果关于iphoneX的人脸识别教程翻译要提上日程了。

年前公司最后一次需求评审大会，PM想要这个效果，类似淘宝天猫京东很多APP下拉转场的效果，当时是被开发拒绝了，而我倒觉得可以试着做一下，然后让它加入[10000ui](https://github.com/blurryssky/10000ui)。这个UI其实年前就做了大概，年后回来一直在做需求。这几天需求做完了，回头来完善了一下它，趁此空挡赶紧更新下博客。

## PullingHeader


![.](PullingHeader.gif)

### 通过两份协议，开发者可以高度自定义下拉动画、刷新和转场条件:

```swift
protocol PullingRefreshing: NSObjectProtocol {
    func shouldRefresh(fraction: CGFloat) -> Bool
    func stateDidUpdate(_ state: PullingState)
}

protocol PullingTransitioning: NSObjectProtocol {
    func shouldTransition(fraction: CGFloat) -> Bool
}
```

`PullingRefreshing `需要让自定义的下拉刷新的view去遵循:

* **shouldRefresh(fraction:)** 这个方法可以去决定下拉到多少百分比就开始刷新，同时可以根据百分比自定义下拉过程中的动画。
* **stateDidUpdate(:)** 根据state变化来更新UI。

 ```swift
enum PullingState {
    case resting(fraction: CGFloat)
    case pulling(fraction: CGFloat)
    case willRefresh(fraction: CGFloat)
    case refreshing(fraction: CGFloat)
    case willTransition(fraction: CGFloat)
    case transitioning(fraction: CGFloat)
}
```

`PullingTransitioning `需要让下拉后转场的view controller去遵循:

* **shouldTransition(fraction:)** 和上面的第一个方法类似，这里只需要在合适的下拉位置来决定是否转场。

### 通过`PullingHeader `对象，来管理下拉过程:

通过上面的协议，`PullingHeader `会自动变化状态，并且在初始化方法中，`PullingHeader `需要得到所有信息。

```swift
init(scrollView: UIScrollView!,
         pullToRefreshView toRefresh: PullingRefreshing!,
         refreshClosure: ((PullingHeader) -> Void)!,
         pullToTransitionViewController toTransition: PullingTransitioning? = nil,
         transitionClosure: ((PullingHeader) -> Void)? = nil) 
```

因为转场不是必要的，所以后面两个参数可以是nil。

> `scrollView、pullToRefreshView、pullToTransitionViewController` 都是`weak`属性

🤔具体的使用方法，可以参照代码，每个UI都有对应的sample view controller去使用它的demo。


## 相关参考资料

### [MJRefresh](https://github.com/CoderMJLee/MJRefresh)

## sample地址

### [10000ui](https://github.com/blurryssky/10000ui)

