---
title: Handling 3D Interaction and UI Controls in Augmented Reality(译)
date: 2018-01-14 17:31:08
tags: 
	- iOS开发 
	- AR
	- ARKit（官方文档翻译）
	
---

## [文档地址](https://developer.apple.com/documentation/arkit/handling_3d_interaction_and_ui_controls_in_augmented_reality)

**🤓这篇文章主要是提供一些提示和示范，来让我们做AR APP的时候增强用户体验。**

## Overview

AR给用户提供新的交互方式。但是，许多人性化交互设计规则依然有效。确保逼真的AR效果也需要精致的3D资源设计和渲染。The iOS Human Interface Guidelines包含了AR的人性化交互规则。这个工程展示了许多方法去应用这些规则，然后创造出沉浸式的AR体验。示例APP提供了一个简单的AR体验，允许用户在真实场景里放置一个或多个虚拟物体，然后用直接的手势去安排他们。APP提供了用户交互线索，来帮助用户明白AR体验的状态和他们的交互选项。

（🙂不重要的话就不翻译了）

## Placing Virtual Objects

帮助用户明白何时去查找一个表面并且防止一个物体。`FocusSquare`类在AR视图里绘制了一个正方形边框，给用户提示ARKit world tracking的状态。

正方形会根据场景深度改变大小和方向，并且用突出的动画在打开和关闭状态之间切换，来表示是否ARKit检测到了适合放置物体的平面。在用户放置物体之后，对焦的正方形会消失,保持隐藏直到用户把摄像头对准到另外的表面。

🤓**在用户放置物体时给予合适的反馈。**当用户选择一个虚拟物体去放置时，sample(示例工程)里的`setPosition(_:relativeTo:smoothMovement)`方法用`FocusSquare`的简单的在屏幕中大概的位置粗略的放置一个物体，即时ARKit还没在那个地方检测到平面。

```swift
guard let cameraTransform = session.currentFrame?.camera.transform,
    let focusSquarePosition = focusSquare.lastPosition else {
    statusViewController.showMessage("CANNOT PLACE OBJECT\nTry moving left or right.")
    return
}

virtualObjectInteraction.selectedObject = virtualObject
virtualObject.setPosition(focusSquarePosition, relativeTo: cameraTransform, smoothMovement: false)

updateQueue.async {
    self.sceneView.scene.rootNode.addChildNode(virtualObject)
}
```

这个位置可能不够精确到用户想放在真实世界的位置，但也足够接近了，而且非常快速。

随着时间，ARKit探测平面和重新预估它们的位置，调用`renderer(_:didAdd:for:)`和`renderer(_:didUpdate:for:)`代理方法来上报结果。在这些方法里，sample调用了`adjustOntoPlaneAnchor(_:using:)`方法来决定是否之前放置的物体接近一个探测到的平面了。如果平面附近的话，用一个很细微的动画把物体移动过去，这样它看起来就在用户选择的位置上了。

```swift
// 如果在附近就移动过去 (5厘米内).
let verticalAllowance: Float = 0.05
let epsilon: Float = 0.001 // 如果小于1mm就不移动了
let distanceToPlane = abs(planePosition.y) // 😓为何用y呢
if distanceToPlane > epsilon && distanceToPlane < verticalAllowance {
    SCNTransaction.begin()
    SCNTransaction.animationDuration = CFTimeInterval(distanceToPlane * 500) // Move 2 mm per second.
    SCNTransaction.animationTimingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)
    position.y = anchor.transform.columns.3.y
    SCNTransaction.commit()
}
```

## User Interaction with Virtual Objects

🤓**允许用户用基本的，熟悉的手势去交互虚拟物体。**sample里用了一根手指tap，一或者两根手指pan，以及两指rotate手势来让用户摆放和转动虚拟物体，`VirtualObjectInteraction`类负责这些手势。

🤓**通常来讲，保持交互简单。**当拖拽一个虚拟物体（见`translate(_:basedOn:infinitePlane:)`），sample APP限制了它只能在二维平面上移动。同样的，因为它是在平面上，旋转手势（见`didRotate(_:)`）只让物体围绕Y轴转动，所以物体仍然在平面上。

🤓**在大概的位置响应可交互的虚体物体的手势。**sample里`objectInteracting(with:in:)`方法根据手势提供的点击区域使用了hit tests。点击虚拟物体的边界时，这个方法让用户的触碰能够影响到虚拟物体，即使触碰区域不在物体的可见区域上。通过给多个手势提供多个hit tests，这个方法让用户的触碰交互更加流畅：

```swift
for index in 0..<gesture.numberOfTouches {
    let touchLocation = gesture.location(ofTouch: index, in: view)
    
    // 寻找是否有物体在触碰位置下
    if let object = sceneView.virtualObject(at: touchLocation) {
        return object
    }
}

// 找不到就把手势中心的物体返回，也可能还是没有
return sceneView.virtualObject(at: gesture.center(in: view))
```

🤓**考虑是否用户创造的对象需要缩放。**AR放置逼真的虚拟物体，它们可能是在用户的世界里自然呈现的，所以保持它原本的大小会增加逼真程度。所以sample没有增加缩放的手势。另外，由于不提供缩放手势，sample防止用户迷惑，用户可能会分不清缩放手势是改变了物体的大小还是改变了物体的远近。（如果你想提供的话，也可以加pinch手势）

🤓**留意潜在的手势冲突。**`ThresholdPanGesture`是一个`UIPanGestureRecognizer`的子类，它提供的效果是，延迟手势的识别效果直到手势已经确实移动并且超过了一个门槛。`touchesMoved(with:)`方法用这个类来让用户平滑的在拖拽和旋转手势之间过渡。

```swift
override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent) {
    super.touchesMoved(touches, with: event)
    
    let translationMagnitude = translation(in: view).length
    
    // 根据手指的数量调整门槛
    let threshold = ThresholdPanGesture.threshold(forTouchCount: touches.count)
    
    if !isThresholdExceeded && translationMagnitude > threshold {
        isThresholdExceeded = true
        
        // Set the overall translation to zero as the gesture should now begin.
        setTranslation(.zero, in: view)
    }
}
```

🤓**保证虚体物体平滑的移动。**sample的`setPosition(_:relativeTo:smoothMovement)`方法在拖拽物体的手势识别的区域和这个物体的历史位置上做了篡改。通过算最近的点的平均位置，这个方法既创造了平滑的移动，也避免了物体被甩在用户的手势后面：


```swift
if smoothMovement {
    let hitTestResultDistance = simd_length(positionOffsetFromCamera)
    
    // 添加最新的位置，并且保证最多纪录10个最近的距离。
    recentVirtualObjectDistances.append(hitTestResultDistance)
    recentVirtualObjectDistances = Array(recentVirtualObjectDistances.suffix(10))
    
    let averageDistance = recentVirtualObjectDistances.average!
    let averagedDistancePosition = simd_normalize(positionOffsetFromCamera) * averageDistance
    simdPosition = cameraWorldPosition + averagedDistancePosition
} else {
    simdPosition = cameraWorldPosition + positionOffsetFromCamera
}
```

🤓**探索更多有趣的手势交互。**在AR体验里，pan代表的是，移动一根手指跨越屏幕，这并不是唯一的自然地拖拽物体到新位置的方式。用户也可能下意识的手指按住物体在屏幕上不动，然后移动设备。（😂）

sample提供这类手势，在拖拽期间不停调用`updateObjectToCurrentTrackingPosition()`方法，即使用户的手势触碰区域改变了。如果在拖拽时设备移动了，这个方法计算了用户触碰区域的新的位置，并且让移动按预期的进行。（😲厉害了我的哥）

## Entering Augmented Reality

🤓**标识出状态指示并且让用户参与进来。**sample通过一个漂浮的文字的视图，展示了AR session的状态文字提示，并且指明了交互建议。`StatusViewController`管理这个视图，在给用户短暂的时间阅读提示后，就会渐变消失，或者是重要的状态信息就保持可见，直到用户去修正了问题。（比如把摄像头移到合适的位置而不是对着墙。）

## Handling Problems

🤓**如果事与愿违允许用户重置或者刷新。**sample添加了一个reset按钮，在屏幕的右上角总是可见，允许用户无视当前状态重启AR。见`restartExperience()`方法。

🤓**只在能用的设备上添加AR功能。**sample需要ARKit作为其核心功能，所以在`Info.plist`的`UIRequiredDeviceCapabilities`里定义了arkit键，这个键防止在不能支持AR的设备上安装该APP。

如果你的APP只是用AR作为一个次要的功能，使用`isSupported`方法来决定是否能展示这些需要ARkit的功能。


