---
title: About Augmented Reality and ARKit(译)
date: 2018-01-11 10:32:35
tags: 
	- iOS开发 
	- AR
	- ARKit（官方文档翻译）
	
---

# [文档地址](https://developer.apple.com/documentation/arkit/about_augmented_reality_and_arkit)

**😋这几天看了几个AR应用，觉得屌爆了，赶快插播ARKit了解一下**

# About Augmented Reality and ARKit

探索支持的概念，特点，以及练习构建一个AR体验。

## Overview

AR的基本要求，同时也是AR的定义，是创建并且跟踪一个在 用户存在的真实世界空间 和 一个你可以模拟虚拟内容的虚拟世界 之间的通信介质。当你的APP把内容和一个动态的相机图片一起展示出来，用户就体验到AR了：是一种你的虚拟内容是真实世界的一部分的错觉。

🤓在所有的AR体验里，ARKit使用世界和相机的坐标系统，沿用right-handed convention:Y轴是指向上方，Z轴指向观看者，X轴指向观看者的右方。

Session configurations可以出于贴近真实世界的原因改变原点和坐标系的方向（参见`worldAlignment`）。每一个AR session里的anchor定义了它自己的坐标系，也是right-handed convention的；比如，`ARFaceAnchor`定义了定位面部特征的坐标系。

## How World Tracking Works

为了创建一个在真实和虚拟空间的通信介质，ARKit用了一种叫做*visual-inertial odometry*的技术。它结合了iOS设备的运动传感器和通过摄像头获取到的场景的计算机图像识别。ARKit识别场景图片里值得注意的特征，在视频帧之间跟踪这些特征的位置的变化，讲信息与运动传感器数据进行对比。结果是高精准度的设备的位置和运动的模型。

🤓World tracking也分析和理解场景的内容。使用hit-testing方法（参见`ARHitTestResult`）来找到摄像头图片里的一个对应真实世界的面的点。如果你在session configuration启用了`planeDetection`，ARKit会检测摄像头图片里的平面，并且上报他们的位置和大小。你可以用 hit-test的结果 或者 `planeDetection`检测到的平面 来放置或者交互你的场景你的虚拟内容。

（初看到这里可能会觉得晕，我举个例子：我想放置一只猫在平面上，那首先需要知道平面的坐标和大小，ARKit有两种办法去获取，一种是主动hit-test，这个和UIResponder的那个不一样，后续会说，另外一种是通过摄像头扫描，代理回调。）

## Best Practices and Limitations

World tracking是一个不精确的技术。它经常能产生感人的精确度，创造逼真的AR体验。但是这依赖于设备的物理环境，而这通常是不连续的或者难以在真实世界测量。（这句翻译不太顺，大概意思就是由于设备硬件是用户自己掌控的，不像软件完全由开发者掌控，所以跟踪的精确度不能保证。）为了创建一个高质量的AR体验，注意以下要点：

* 建立良好的光线条件。World tracking涉及图片分析，这要求图片清晰。跟踪的质量会因为摄像头没法看清细节而下降，比如摄像头被指向一片空白的墙或者场景太暗。

* 使用跟踪质量信息给用户反馈。World tracking关联图片分析和设备运动。如果设备移动，ARKit可以对场景有更好的理解，即使是很小的移动。过度的运动，太远，太快，或者摇晃的太厉害，都会造成模糊的图片，或者造成视频帧之间的跟踪特征距离太远，从而降低跟踪质量。`ARCamera`提供跟踪状态信息，你可以用来开发UI来告诉用户如何解决跟踪质量低的状况。

* 花时间给plane detection来创造清晰的结果，然后在得到需要的结果后关闭plane detection。plane detection的结果随着时间变化，一个平面第一次被探测到，它的位置和大小可能不精确。随着时间平面停留在场景里，ARKit会改善它的位置和大小的估量。当一个大平面在场景里，在你已经放置了内容的的平面上，ARKit可能会持续的改变plant anchor的位置，大小以及transform。（所以这是需要检测完毕后关掉plane detection的原因？😑，这文档感觉话说一半就完了。。）
