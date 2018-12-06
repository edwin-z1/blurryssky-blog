---
title: PetalLayout
date: 2018-01-30 16:48:44
tags:
	- iOS开发 
	- custom UI
	
---

# 诞生原因

😅我的天，我真是到处挖坑，先列一个未完成列表和原因：

* 面试题和知识总结，这个还没开始，主要是想直接研究新知识。

* CoreAnimation（官方文档翻译)，翻译了一点，和第一个原因差不多，我对CoreAnimation也比较熟悉，想花时间在陌生的地方，但是我一定会更完的。

* ARKit（官方文档翻译），这个官方文档差不多了，还有一篇文章是iphoneX才能用的，没有设备暂时不译了；其他关于AR的技术，比如空间画笔，近期会总结。

关于*custom UI*分类，我本身一直在Github上维护着一个[10000ui](https://github.com/blurryssky/10000ui)的*swift*代码库。

这几天工作的时候遇到一个需求可能会需要去修改一下`UICollectionView `的布局，就去研究了一下`UICollectionViewLayout `这个类。其实以前做过一个像花瓣一样从中心绽开的选择菜单，当时是用按钮和循环做的，这次就索性用`UICollectionViewLayout`的子类去重构一次，顺便学习新知识。

# PetalLayout


![.](PetalLayout.gif)

## sample里我实现了三种效果:

* 依次插入删除

* 随机位置随机个数插入删除

* 依次多个插入删除


强裂建议大家先看下方的参考资料，把握每个细节。这里我简单介绍下:

`UICollectionViewLayoutAttributes`，首先需要说一下这个类，这个类用于存放cell的信息，比如frame、alpha，系统会自己在合适的时候为cell赋值，我们也是通过这个类和系统的`UICollectionView`传递信息。

`UICollectionViewLayout`是一个抽象类，本身无法使用，它是用于管理collection view里的cell、supplementary view、decoration view的`UICollectionViewLayoutAttributes`信息，使用方法就是去继承它，然后重写以下方法:(系统也为我们实现了一个`UICollectionViewFlowLayout`，方便快速是哟并)

* **collectionViewContentSize** 这个方法会决定scroll view的滑动大小，自己去计算布局里的所有内容大小就好。
* **prepare** 每次更新都会走这个方法，比如*reloadData、insert、delete*等操作，可以在这里预先配置好位置信息。
* **layoutAttributesForElements(in:)** 决定显示哪些cell，节省内存你懂得吧。
* **layoutAttributesForItem(at:)** 有需要时系统会在*reloadData、insert、delete*等操作时询问某个indexPath的布局信息，所以需要实现这个方法。

以上都是必须实现的方法。

关于*insert、delete*动画，需要额外重写以下方法:

* **initialLayoutAttributesForAppearingItemAtIndexPath** 询问某个indexPath的起始位置。
* **finalLayoutAttributesForDisappearingItemAtIndexPath** 询问某个indexPath的终点位置。

注意一点，我们之前的方法已经实现了item的默认位置，也就是没有动画的时候item被摆放的位置。

在*insert*时，item是从`initialLayoutAttributes...`到默认位置。

在*delete*时，itemn从默认位置到`finalLayoutAttributes...`。


🤔PetalLayout的代码只有200行，但是我依然把它分类到*Complex*下，因为我觉得这个含金量还是比较高的，我写的还是有瑕疵(在随机多个插入删除的时候)，但是修改起来我觉得比较困难，希望厉害的同学给我提*pull request*吧。

🤓如果看完觉得很疑惑，一定要看下面的几篇资料。

## 相关参考资料

### [官网文档](https://developer.apple.com/documentation/uikit/uicollectionviewlayout)

### [objc.io(issue-3-3)](https://objccn.io/issue-3-3)

### [raywenderlich(pinterest)](https://www.raywenderlich.com/164608/uicollectionview-custom-layout-tutorial-pinterest-2)


## sample地址

### [10000ui](https://github.com/blurryssky/10000ui)



