---
title: Core Animation Basics(译)
date: 2018-01-04 16:44:52
tags: 
	- iOS开发 
	- CoreAnimation
	- CoreAnimation（官方文档翻译）
	
---

# [文档地址](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html)

**😑有些翻译的不通顺的地方，一定是我还没理解透，有些口水话直接不翻译了**

# Core Animation Basics

CoreAnimation提供一种普遍的机制来给你的APP做动画。它不是用来替换view的，而是一种结合view，让view的性能更好并且动起来的一种技术。它主要是通过缓存view的内容到bitmap，bitmap是可以直接被图形硬件处理的。有时候，这可能需要你去重新思考怎么表现和管理你的APP的内容，但大多数时候你都不知道你正在使用CoreAnimation。除了缓存view的内容，CoreAnimation也可以让你制定任意的内容，把它结合在你的view里，并且一样的能够动画。

使用CoreAnimation来让APP的view和其它可见对象动起来。大部分改变关联到修改这些可见对象的属性。比如，你可以用CoreAnimation来改变view的position, size, opacity。当你改变任意一种时，CoreAnimation在这个属性的当前值和新值之间做动画。一般来说你不会用CoreAnimation去改变一个view的内容，一秒60次，像动画一样。而是用CoreAnimation来移动view的内容，淡入淡出，或者任意的图形变换，或是别的可视性属性。


## Layers Provide the Basis for Drawing and Animations

layer对象是3D空间里的2D表示对象，它是CoreAnimation的核心。和view相同的是，layers管理几何信息，内容，其自身的视图属性。和view不同的是，layers不去定义它自身的外形。layer只是去管理其bitmap的状态信息。bitmap可以是一个view绘制自身的结果或者是一张你指定的图片。因此，在App里使用的layer主要是当做model使用，因为他们主要是管理数据。这个概念很重要需要铭记，因为它会影响到动画行为。


### The Layer-Based Drawing Model

大部分layer实际上不会去绘制。而是捕获其content，然后把它们缓存到一个bitmap，有时候也被叫做*backing store*。当频繁地更改layer的属性的时候，其实是在改变其状态信息。当改变触发动画时，CoreAnimation传递layer的bitmap和状态信息给GPU，GPU用新的信息渲染bitmap。在GPU上操作bitmap产生的动画比软件上能做的快得多。

因为CoreAnimation操作静态的bitmap，基于layer的绘制不同于基于view的绘制技术。基于view的绘制，改变view经常会触发到view的`drawRect:`方法来用新的参数重绘。但是这消耗很大，因为是在CPU和主线程上完成的。CoreAnimation尽可能的通过操作缓存在硬件上的的bitmap来避免这种开销，并且达到相同或类似的效果。

虽然CoreAnimation尽可能的用缓存的内容，你的APP仍然需要提供初始内容并且随时更新。

### Layer-Based Animations

一个layer对象的数据和状态信息是和layer在屏幕上呈现的图像内容解耦的。这种解耦使得CoreAnimation处于在旧状态动画到新状态之间的中间部分。比如，改变layer的position，使CoreAnimation从现在的position移动到新指定的position。类似的改变其他属性也引起合适的动画。

在动画过程中，CoreAnimation在硬件上做了每一帧的绘制。你只需要指定动画的开始和结束，其它的交给CoreAnimation。你也可以指定其它参数，如果没有指明，CoreAnimation也会提供合适的默认值。

## Layer Objects Define Their Own Geometry

layer的职责之一是管理其内容的视觉几何关系。包括bounds, positon, 以及是否layer被旋转，缩放，或别的变换。类似view, layer有frame和bounds， layer还有一些view没有的，比如anchor point。

### Layers Use Two Types of Coordinate Systems

（这里主要是说iOS和OS X的坐标系统不同，略过）

### Anchor Points Affect Geometric Manipulations

（这里主要讲anchor point，略过）

### Layers Can Be Manipulated in Three Dimensions

每个layer有两种你可以使用的transform矩阵。`CALayer`的transfrom属性会影响到它和它的sublayers。一般都用这个属性来修改layer本身。比如，你可以用它来缩放或旋转或改变position。`sublayerTransform `这个属性会额外的影响到sublayers，它主要用来给layer的内容增加透视效果（3D效果）。

transform通过给矩阵上的数字进行坐标乘算来得到新的坐标来表示变换后的点。由于CoreAnimation的值可以在三维里被指定，每个坐标点有4个值必须被4X4的矩阵乘算。在CoreAnimation里，图形的transfrom被表示成`CATransform3D `类型。幸运的是，你不必直接取修改这种结构里的值来完成一些基础变换。CoreAnimation提供了一些列方法来创建scale, translation, rotation等等。除此之外，你也可以用`key-value coding`来修改transform。详建[CATransform3D Key Paths](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Key-ValueCodingExtensions/Key-ValueCodingExtensions.html#//apple_ref/doc/uid/TP40004514-CH12-SW1)。

## 🤓Layer Trees Reflect Different Aspects of the Animation State

使用CoreAnimation的APP有三组layer对象。每一组都扮演不同的角色：

在model layer tree的对象（也可简称"layer tree"）是你交互的最多的。这些对象是存储任何动画的目标值的model。当你改变一个layer的属性时，你就使用了这些对象中的一个。

在presentation tree上的对象包含了正在执行的动画过程中的值，它反映屏幕上layer属性的当前值。你永远也不应该修改这个树上的对象。而是只用它来读取动画的当前值，或许基于当前值来创建一个新的动画。

在render tree上的对象是动画的实际执行者，并且是CoreAnimation私有的。

每一组layer对象都像view一样被组织分层。实际上，在一个所有view都有layer的APP上（OS X上不一定所有view都有layer，但iOS一定有），初始的树的结构和view的层级结构一模一样。然而，一个APP可以增加额外的layer，所以，layer之后不一定后view的层级一样。（三组layer的层级结构和view类似，由于layer可以直接被添加，所以layer的层级可能会更多）

**重要：你应该只在动画进行过程中访问presentation tree的对象，它会反映当时屏幕上动画的属性的值，之后就会和model layer tree的值一样**

## 🤓The Relationship Between Layers and Views

layer不是用来替换APP里的view的，你不能仅凭layer来创建一个可视化接口。layer是view的基础设施。明确地说，layer让绘制和动画更简单更高效。然而，有很多事layer做不到。layer不能处理事件，绘制内容（`drawRect`），参与到响应者链，或者别的很多事。因此，每个APP还是应该有一个或多个view来处理交互事件。

在iOS，每一个view都有layer，而OS X上只是大多数是, layer确实会占用更多内存，但是它们带来的优势通常比劣势好很多。

在layer-backed的view里，系统负责创建layer并且和view同步。（其它和OS X有关）

注意：layer-backed的view，推荐尽可能直接去操作view而不是layer。在iOS，view是layer的一个轻便的解包对象，所以你对layer直接操作而不去操作view，大多数时候是ok的。但有时候可能就会出问题。这篇文档指出了那些陷阱（？在哪）并且希望给你提供正式的使用方式。
除了使用layer-backed的view，你也可以直接创建layer来使用，这也是一种性能优化方式，比如，你想用一张图片在许多地方展示，你可以加载图片一次，然后赋值给许多独立的layer，这些layer会持有同一份图片，但不会在内存里赋值一份。
