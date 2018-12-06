---
title: Creating Face-Based AR Experiences(译)
date: 2018-03-26 14:17:11
tags: 
	- iOS开发 
	- AR
	- ARKit(官方文档翻译）
	
---

# [文档地址](https://developer.apple.com/documentation/arkit/creating_face_based_ar_experiences)

**😋使用ARKit来做人脸识别，一切变得简单不少，还可以使用超多的脸部细节**


## Overview

文档上有sample app下载，它提供了一个简单的UI，允许我们在有TrueDepth前置摄像头的设备上(目前只有iphoneX)上切换选择4种AR效果。

* 无任何AR效果的摄像
* 由`ARKit`提供的面部网膜，并且自动预估真实世界的方向光线环境。
* 在用户面部贴上虚拟的3D内容（3D内容可以被用户的脸遮挡）。
* 一个会跟着用户面部变化而动画的简单机器人。

## Start a Face Tracking Session in a SceneKit View

像其他的`ARKit`用法一样，脸部跟踪需要配置和启动`ARSession`对象，然后把摄像图片和虚拟内容在一个*view*上渲染。关于更多的设置*session*和*view*的细节,请看[About Augmented Reality and ARKit]()、[Building Your First AR Experience]()。这里使用`SceneKit `来展示AR体验，也可以用`SpriteKit `或用`Metal`打造自定义的渲染器，请看（[ARSKView and Displaying an AR Experience with Metal]()）。启用面部跟踪，需要创建一个`ARFaceTrackingConfiguration `实例，配置它的属性，把它传有*view*关联的`ARSession`对象的**run(_:options:)**方法，示例如下。

```swift
guard ARFaceTrackingConfiguration.isSupported else { return }
let configuration = ARFaceTrackingConfiguration()
configuration.isLightEstimationEnabled = true
session.run(configuration, options: [.resetTracking, .removeExistingAnchors])
```

记得使用**ARFaceTrackingConfiguration.isSupported**来检查设备是否支持。

## Track the Position and Orientation of a Face

面部追踪启用时，`ARKit`自动为运行的*AR session*添加`ARFaceAnchor `对象，这个对象包含了用户脸部的信息，包括位置和方向。

> `ARKit `只给探测和提供一个用户的信息。如果多张脸在摄像头内，`ARKit`选择最大最清晰的那张脸

在基于`SceneKit`的AR体验里，你可以在对应的*face anchor*里用**(_:didAdd:for:)**添加3D内容。`ARKit `为这个*anchor*添加一个`SceneKit `*node*，并且在每一帧更新*node*的位置和方向，所以你给这个*node*添加的任何`SceneKit `内容自动的跟随用户脸部的位置和方向。

```swift
func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor) {
    // 持有`faceNode`，这样在切换内容时不必重启*session*。
    faceNode = node
    serialQueue.async {
        self.setupFaceNodeContent()
    }
}
```

在**renderer(_:didAdd:for:)**方法里调用了**setupFaceNodeContent**方法来为`SceneKit `内容添加**faceNode**。如果你在代码里改变**showsCoordinateOrigin**变量的值，会在*node*上增加一个可见的x/y/z坐标轴，标识*face anchor*坐标系的原点。 


## Use Face Geometry to Model the User’s Face

`ARKit `提供一个粗略的匹配用户脸部的大小、形状、拖布结构、当前表情的3D网状几何*geometry*。`ARKit `也提供
`ARSCNFaceGeometry `类，提供一种简单的方式来可视化这个网。

你的AR应用可以用这个网来放置或者绘制贴脸内容。比如，可以提供一个半透明的纹理到这个*geometry*上，你可以在用户皮肤上画虚拟的纹身或者化妆。

为了创建一个`SceneKit `脸部几何，初始化一个`ARSCNFaceGeometry `对象:

```swift
// This relies on the earlier check of `ARFaceTrackingConfiguration.isSupported`.
let device = sceneView.device!
let maskGeometry = ARSCNFaceGeometry(device: device)!
```

**setupFaceNodeContent**方法给*scene*添加了一个包含*face geometry*的*node*，也就是一个面部。由于这个*node*是被回调方法提供的*face node*的*child node*，所以它会自动的跟踪用户脸部的位置和方向。但是为了也让它跟踪用户脸部的形状，包括眨眼、讲话、各种面部表情，你需要在**renderer(_:didUpdate:for:)**更新它的*geometry*。

```swift
func renderer(_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor) {
    guard let faceAnchor = anchor as? ARFaceAnchor else { return }
    
    virtualFaceNode?.update(withFaceAnchor: faceAnchor)
}
```

```swift
func update(withFaceAnchor anchor: ARFaceAnchor) {
    let faceGeometry = geometry as! ARSCNFaceGeometry
    faceGeometry.update(from: anchor.geometry)
}
```


## Place 3D Content on the User’s Face

另一个使用`ARKit `提供的面部网络的用法是创建一个*occlusion geometry*(😦这个有点抽象，我确实不知道怎么翻译)。*occlusion geometry*是一个3D模型，它不渲染任何可见内容（允许相机图片穿过），但是阻断了其他虚拟内容的相机的*view*。

这个技巧创造一种脸部和虚拟对象交互的假象，即使脸是一个2D相机图片，而虚拟内容是一个被渲染的3D对象。比如，如果你放置一个*occlusion geometry*和虚拟墨镜在用户脸上，脸可以挡住墨镜的框架。

为了给脸创建一个*occlusion geometry*，从创建一个`ARSCNFaceGeometry `对象开始，参照之前的例子。然而，代替设置给对象的*material*一个可见的外形的是，设置*material*的渲染depth:

```swift
geometry.firstMaterial!.colorBufferWriteMask = []
occlusionNode = SCNNode(geometry: geometry)
occlusionNode.renderingOrder = -1
```

因为渲染了*depth*，其他被渲染的对象正确的出现在了它的前面或者后面。由于它没有渲染颜色，相机图像也出现了。sample里结合了这个技巧和`SceneKit `对象，放置在了用户眼前，制造了一种真实的被用户的鼻子遮挡的图像。

（其实不太理解原理）

## Animate a Character with Blend Shapes

除了上面的例子外，`ARKit `也提供一种更抽象的用户面部表情模型，用一个`blendShapes `字典的形式。你可以用字典里的多个值去控制你的2D或者3D资源的动画参数，创建一个特征（比如头像或者木偶），跟随用户的面部移动和表情。

作为一个*blend shape*的基础示例，这个sample包括了一个简单的机器人头部模型，是用`SceneKit `的基础形状创造的。（在*robotHead.scn*查看）

为了获取用户当前的面部表情，在**renderer(_:didUpdate:for:)** 里读取*face anchor*的`blendShapes `字典：

```swift
func update(withFaceAnchor faceAnchor: ARFaceAnchor) {
    blendShapes = faceAnchor.blendShapes
}
```

然后，检查字典的键值对来计算动画参数。这里有52个独一无二的`ARFaceAnchor.BlendShapeLocation`参数。你的app可以尽可能少也可以尽可能多的使用它们来创造你想要的艺术效果。在这个示例工程里，`RobotHead `完成了这个计算，转换`eyeBlinkLeft `和`eyeBlinkRight `参数到机器人眼睛的大小参数，用`jawOpen `来偏移机器人下巴。

```swift
var blendShapes: [ARFaceAnchor.BlendShapeLocation: Any] = [:] {
    didSet {
        guard let eyeBlinkLeft = blendShapes[.eyeBlinkLeft] as? Float,
            let eyeBlinkRight = blendShapes[.eyeBlinkRight] as? Float,
            let jawOpen = blendShapes[.jawOpen] as? Float
            else { return }
        eyeLeftNode.scale.z = 1 - eyeBlinkLeft
        eyeRightNode.scale.z = 1 - eyeBlinkRight
        jawNode.position.y = originalJawY - jawHeight * jawOpen
    }
}
```

