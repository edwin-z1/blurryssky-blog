---
title: 《App架构》读书笔记（二）MVC
date: 2018-12-3 10:05:04
tags: 
---

**(使用的序号对应的是书中的章节，但没有把所有章节都列出来)**

# 3.Model-View-Controller

>MVC的核心思想是，controller层负责将model层和view层撮合到一起工作。Controller对另外两层进行构建和配置，并对model对象和view对象之间的双向通讯进行协调。所以，在一个MVC app中，controller层是作为核心来参与形成app的反馈回路的:

![1](1.png)

1图和我们上在上一篇文章中的图相比，多了一些更改内部状态的箭头，这其实更加符合真实的情况。

MVC基于经典的面向对象原则:

- 对象在内部对它们的行为和状态进行管理，并通过类和协议的接口进行通讯。
- view对象通常是有一些*view state*需要即时更新的，而不是等到model对象确认修改后再作更新。
- model对象独立于表现形式之外，且避免依赖程序的其他部分。
- 而将两部分组合起来成为完整的程序，则是controller层的责任。

---

MVC的两个缺点：

- 在上一篇文章里我也有提到，由于真实的开发情况里，大家对步骤（3）并没有完成的很好，有时候view对象只需要一次model的信息就不做更改了，甚至没有步骤（3）出现的需要；而在view对象需要依赖model对象完成不断地更新的时候，由于大家可能没有严格遵守单向数据流，也很容易产生数据和UI展示不同步而产生的bug，或者说为了修复bug，产生了很多杂乱的代码。

- MVC的第二个缺点是众所周知的，就是controller层拥有太多的职责的问题，我们将它戏称为 “view controller 肥大化” (massive view controller，它的缩写和 MVC 模式相同)。但是就我看来，我一直不认为这是一个难以逾越的问题，在你明白MVC的意义之后，你才能做出改进，对controller甚至model和view进行更多的分层，而不是把所有的代码都扔进controller。

## 观察者模式失效

>model和view的同步可能失效。当围绕model的观察者模式没有被完美执行时，这个问题就会发生。常⻅的错误是，在构建 view时读取了model的值，而没有对后续的通知进行订阅。另一个常⻅错误是在变更model的同时去更改view，这种做法假设了变更的结果，而没有等待model进行通知，如果model拒绝了这个变更的话，就会发生错误。这类错误会使得view和model 不同步，奇怪的行为也随之而来。

需要说明的是，这里的观察者模式并不是单纯指的`KVO`，它泛指一切接收变更的框架技术，比如`Notification`和`RxSwift`等。同样的通知并不代表的仅是`Notification`，而是一切能够发出变更信息的框架技术，比如`delegate`等。

改进的办法其实很简单。约束自己总是遵守1图中的单向数据流原则，并且使用好用的框架工具。

## 肥大的view controller

>非常大的view controller通常进行了它们的主要工作 (观察model，展示view，为它们提供数据，以及接收view action)之外的无关工作; 它们要么应该被打散成多个各自管理一个较小部分的view层级的controller; 要么就是因为接口和抽象没能将一段程序的复杂度封装起来，view controller做了太多打扫垃圾的工作。

改进的办法是将细小的功能继续分离，把获取数据和处理等方法移到model层，把尽可能多的属于view自身变更的方法移到view层，除此之外，对于剩下的view controller，还可以继续进行以下办法的分离。

### 有局限性的Storyboard

我个人还是很支持使用storyboard，但是它确实有一些缺点，比较明显的一个是，没有办法在初始化时设置好对model的依赖，所以我们可以这样做

```swift

extension FolderViewController {
  static func instantiate(_ folder: Folder) -> FolderViewController {
    let sb = UIStoryboard(name: "Main", bundle: nil)
    let vc = sb.instantiateViewController(withIdenti􏰊er: "folderController") as! FolderViewController 
    vc.folder = folder
    return vc
    } 
}

```

把初始化方法封装起来，减少在view controller里创建的代码，也能防止因为忘记传值而出错。

### 在扩展里进行代码重用

>要在不同的view controller间共享代码，一个常⻅的方法是创建一个包含共通功能的父类。然后view controller就可以通过子类来获得这些功能了。这种技术可以工作，但是它有一个潜在的不足:我们只能为我们的新类选定单个父类。比如说，我们不能同时继承UIPageViewController和UITableViewController。
>
---
>
>这种方式还经常会导致我们常说的**上帝view controller**的问题:一个共享的父类包括了项目中全部的共享的功能。这样的类通常会变得非常复杂，难以维护。
>
---
>
>在view controller中共享代码的另一种选择是使用扩展。在多个view controller中都出现的方法有时候能够被添加到 UIViewController的扩展中去。这样一来，所有的view controller就都能获取这个方法了。

在这个程度之上，为了更加有区别的对待某些功能，我们还可以把它们归类到协议里，然后在协议扩展里实现这些方法。这样view controller在需要某些功能时，必须显式的指定遵循某个协议。这就像是多继承的使用方法，并且由于有显式的声明，能让开发者能清楚地了解这个view controller得到了哪些功能。

不仅是view controller，对于所有的类，都可以这样做。

### 利用Child View Controller进行代码重用

Child view controller是在view controller之间共享代码的另一种选项。有些界面可能会被多次使用，并且自身也包含了许多逻辑，我们可以将它作为child view controller在不同的页面里嵌套。甚至有些时候，一个页面可能过于复杂，我们也可以将它的分类，然后把单一的职责划分给多个child view controller，让parent view controller来管理它们，这样很合理的把代码划分到了不同的文件里。

### 提取对象

我举个例子，在一个编辑音视频的view controller里，会有大量的代码和`AVFoundation`打交道来做媒体方面的逻辑，而且通常这些逻辑代码会很长很多，这个时候可以把他们提取出来，把接口暴露出来成为一个工具类，它可以是只为这个view controller服务，也可以在别的页面公用，这取决于如何设计接口。

### 简化View配置代码

这个其实很常见了，就是把设置和更新view的代码都归纳到view里去，比如传入一个model，然后对自身的UI元素做一系列的更新。

## 总结

>MVC是我们在本书中所讨论的最简单，也是最常用的模式。本书中所展示的其他所有的app设计模式，或多或少都是对惯例的破坏。除非你确实知道你想要为你的项目选择一个不那么通俗的路径，否则你可能还是应该使用MVC来开始你的项目。
>
>---
>
>在一个程序员表达对某个不同架构模式的拥护时，他们有时候会贬低MVC。对于MVC的负面评论包括:无法测试的view controller，view controller会增⻓得过大而且无法控制，数据依赖难以管理等。但是，在阅读本章后，我们希望你意识到，虽然MVC确实有它自己的挑战，但是写出清晰简洁，并包含完整测试以及明确对数据依赖进行管理的view controller，是完全可能的。

这也是我一直持有的态度，MVC非常灵活好用，要理解它的意义，而不是进行非常简单的代码分层，然后把90%的代码都扔进view controller里，上面说的几种方法应该灵活使用。除此之外，一个项目不一定非要只使用一个架构，有些子页面也完全可以采取别的架构，这并不冲突！