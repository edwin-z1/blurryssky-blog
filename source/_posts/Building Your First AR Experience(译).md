---
title: Building Your First AR Experience(译)
date: 2018-01-11 14:26:13
tags: 
	- AR
	
---

# [文档地址](https://developer.apple.com/documentation/arkit/building_your_first_ar_experience)

**😋要做一个AR效果，比如把一只小火龙放在桌子上，我们依靠`SceneKit`或者`SpriteKit`来提供小火龙虚拟图像，依靠`ARKit`为我们提供桌面的位置和大小。**

创建一个使用AR session，并且使用plane detection来放置`SceneKit`3D内容的APP。
(建议去文档地址里把DEMO APP下载下来，先装上试试看，以便理解。)

## Overview

这个实例APP使用`ARKit`world tracking session，把内容展示在`SceneKit`的view里。为了展示plane detection，APP简单的放置了一个`SCNPlane`对象来可视化探测到的`ARPlaneAnchor`对象。

## Configure and Run the AR Session

`ARSCNView`是一个`SceneKit`view,包含一个`ARSession`对象，用来管理为了创造AR体验的运动跟踪和图片处理。另外，为了运行session,你必须提供一个session configuration。

`ARWorldTrackingConfiguration`提供高精度的运动追踪，提供特征来帮助你放置虚拟内容在真实世界的表面上。要运行一个AR session，创建一个包含选项的session configuration对象（比如plane detection），然后对在`ARSCNView `view上的session对象调用`run(_:options:)`方法：

```swift
let configuration = ARWorldTrackingConfiguration()
configuration.planeDetection = .horizontal
sceneView.session.run(configuration)
```

只在view将被显示在屏幕上的时候运行你的seesion。
> 如果你的APP用ARKit作为核心功能，在你的APP的`Info.plist`的`UIRequiredDeviceCapabilities`栏里添加arkit，这样可以避免不支持你的APP的设备安装此APP。如果AR只是次要的功能，使用`isSupported`属性来决定是否提供AR功能。

## Place 3D Content for Detected Planes

在设置好你的AR session后，你可以用`SceneKit`在view里放置内容。
在启用plane detection后,`ARKit`为每一个探测到的平面添加和更新anchors。默认的，`ARSCNView`会为每一个anchor在`SceneKit`场景里添加一个`SCNNode`。你的view的代理可以实现`renderer(_:didAdd:for:)`方法来增加内容到场景里。

```swift
func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor) {
       // 通过plane detection设置后发现的anchor都是`ARPlaneAnchor`
       guard let planeAnchor = anchor as? ARPlaneAnchor else { return }
       
       /*
       创建一个`SceneKit`plane来可视化plane anchor,用到了它的position和extent
       （extent是大小的意思，注意坐标系里Y是向上的，而X和Z是水平的，所以`ARPlaneAnchor`只赋值了X和Z）
       */
       let plane = SCNPlane(width: CGFloat(planeAnchor.extent.x), height: CGFloat(planeAnchor.extent.z))
       let planeNode = SCNNode(geometry: plane)
       planeNode.simdPosition = float3(planeAnchor.center.x, 0, planeAnchor.center.z)
       
       
       // `SCNPlane`对象在它本身的坐标系里是竖直放置的，所以旋转它，让它的方向到水平上来
       
       planeNode.eulerAngles.x = -.pi / 2
       
       // Make the plane visualization semitransparent to clearly show real-world placement.
       planeNode.opacity = 0.25
       
       /*
       把创建好的`planeNode`添加到`ARKit`管理的node上，这样`ARKit`就能随着plane的预估持续跟踪
       在plane anchor上的它的改变
       */
       node.addChildNode(planeNode)
}
```

如果你给anchor对应的node增加内容，作为它的cvhild，`ARSCNView`会随着`ARKit`改善对plane的位置和大小的预估，自动的移动那部分内容。为了展示预估的plane的全部大小，示例APP也实现了`renderer(_:didUpdate:for:)`方法，更新`SCNPlane`对象的大小来显示`ARKit`的预估。

```swift
func renderer(_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor) {
    // 确保更新的是`renderer(_:didAdd:for:)`里添加的那个plane node.
    guard let planeAnchor = anchor as?  ARPlaneAnchor,
        let planeNode = node.childNodes.first,
        let plane = planeNode.geometry as? SCNPlane
        else { return }
    
    // 平面的预估可能会移动平面的中点
    planeNode.simdPosition = float3(planeAnchor.center.x, 0, planeAnchor.center.z)
    
    /*
    平面预估可能会扩大平面的大小，或者结合之前探测到的平面变成更大的。
    在后者，`ARSCNView`自动的删除每个plane对应的node，然后调用这个方法更新大小。
    （不太懂，所以翻译不太顺，不过这里问题不大。😅）
    */
    plane.width = CGFloat(planeAnchor.extent.x)
    plane.height = CGFloat(planeAnchor.extent.z)
}
```