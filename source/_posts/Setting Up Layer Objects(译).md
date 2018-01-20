---
title: Setting Up Layer Objects(译)
date: 2018-01-07 15:31:19
tags: 
	- iOS开发 
	- CoreAnimation
	- CoreAnimation（官方文档翻译）
	
---

# [文档地址](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/SettingUpLayerObjects/SettingUpLayerObjects.html)

**😑有些翻译的不通顺的地方，一定是我还没理解透，有些口水话直接不翻译了**

# Setting Up Layer Objects

layer是CoreAnimation的核心。layer管理APP的视觉的内容，提供给改变内容风格和视觉外形的选项。iOS APP已经自动打开了layer支持，OS X APP必须显示指定打开支持。一旦启用，你需要知道如何配置和操作这些layer。

## Enabling Core Animation Support in Your App

(主要针对OS X开发者，略)

## Changing the Layer Object Associated with a View

layer-backed view默认创建一个CALayer类型的对象，大多数时候你也不需要别的layer类型的对象。但是CoreAnimation提供不同的layer类，每一种都提供一些特别的功能。选择一个不同的layer可能会增强你的APP体验或者简单的完成一种特别的功能。比如，CATiledLayer是用来优化显示打图片的。

### Changing the Layer Class Used by UIView

你可以重写`layerClass`方法来返回一个不同的类对象。大多数iOS上的view创建一个CALayer对象并且用它来作为它的内容的backing store。大多数你自己的view，这个默认选择是好选择，你应该不要去改变它。但是某些情况你可能会发现另外的layer类更合适。比如，你可能想在下列情况来改变layer的类:

* 你的view通过`Metal`或者`OpenGL ES`来绘制内容，所以你会使用`CAMetalLayer `或`CAEAGLLayer `。

* 有一种更针对你的使用场景的layer，可以提供更好的体验。

* 你想要利用一些类的功能，比如`particle emitters`或`replicators `。

想要改变类的backed layer很简单。只需要重写`layerClass`方法来返回一个不同的类对象，在显示之前，view会调用`layerClass`方法，并且使用返回的类来创建layer对象。一旦创建，view的layer就不能再改了。

```swift
+ (Class) layerClass {
   return [CAMetalLayer class];
}
```

在原文文档中你可以看到所有不同的layer类。[Different Layer Classes Provide Specialized Behaviors](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/SettingUpLayerObjects/SettingUpLayerObjects.html#//apple_ref/doc/uid/TP40004514-CH13-SW25)

## Providing a Layer’s Contents

layer是管理APP内容的数据对象。一个layer的内容由一个包含你想展示的图形数据的bitmap组成。有以下三种方式为bitmap提供内容：

1. 直接赋值一个图片对象给layer的`contents`属性。（适用于layer的内容不变，或者很少变的情况。）

2. 给layer赋值一个`delegate`对象并且让delegate来绘制内容。（适用于layer的内容会经常变化，并且会依赖一个外部对象，比如view。）

3. 定义一个layer子类，并且重载绘制方法来自己提供内容。（适用于你必须创建一个自定义子类或你想改变原本的绘制行为。）

以上说的这三种方法，只在你自己创建layer的时候需要考虑到。如果你的APP只是包含layer-backed view，这些都不用考虑。layer-backed view自动为他们的layer用最高效的方式提供内容。

### Using an Image for the Layer’s Content

因为layer只是一个管理bitmap图的容器，你可以直接给layer的`contents`属性赋值一个图片。赋值图片给layer很简单，它会直接饮用这个图片对象，不再创建copy对象。这个行为可以节省内容消耗。

赋值给layer的图片必须是`CGImageRef `类型。当赋值图片时，记得提供适合设备的分辨率。在retina屏上，这可能要求你去调整图片的`contentsScale `。更多信息，后续会提到。

### 🤓Using a Delegate to Provide the Layer’s Content

如果你的layer的内容动态变化，你可以用`delegate`来提供和更新内容。在显示时，layer调用代理方法来提供需要的内容：

* 如果`delegate`实现了`displayLayer:`方法，那么代码实现负责创建bitmap并且赋值给layer的`contents`属性。

* 如果`delegate`实现了`drawLayer:inContext:`方法，CoreAnimation创建bitmap，创建graphics context来绘制到bitmap里，然后调用这个代理方法来填充bitmap。所以代码实现需要绘制到提供的context里。

`delegate`必须实现上面两种方法之一。如果都实现了，只会调用`displayLayer:`方法。

重载`displayLayer:`是优先考虑的，在下面的例子中，`delegate`用一个helper对象来加载和显示图片，并且根据内部的`displayYesImage`BOOL值判断显示哪一张图片：

```swift
- (void)displayLayer:(CALayer *)theLayer {
    // Check the value of some state property
    if (self.displayYesImage) {
        // Display the Yes image
        theLayer.contents = [someHelperObject loadStateYesImage];
    }
    else {
        // Display the No image
        theLayer.contents = [someHelperObject loadStateNoImage];
    }
}
```

如果你没有图片或者helper对象来创建bitmap，你也可以用`drawLayer:inContext: `自己绘制，在下面的例子中，`delegate`绘制了一个简单的弧线：

```swift
- (void)drawLayer:(CALayer *)theLayer inContext:(CGContextRef)theContext {
    CGMutablePathRef thePath = CGPathCreateMutable();
 
    CGPathMoveToPoint(thePath,NULL,15.0f,15.f);
    CGPathAddCurveToPoint(thePath,
                          NULL,
                          15.f,250.0f,
                          295.0f,250.0f,
                          295.0f,15.0f);
 
    CGContextBeginPath(theContext);
    CGContextAddPath(theContext, thePath);
 
    CGContextSetLineWidth(theContext, 5);
    CGContextStrokePath(theContext);
 
    // Release the path
    CFRelease(thePath);
}
```

对于自定义内容的layer-backed views，你应该重载`view`的方法来绘制。一个layer-backed view自动的成为它的layer的代理并且实现所需要的方法，不要去改变。而是去重写view的`drawRect: `方法。

### 🤓Providing Layer Content Through Subclassing

如果你一定要实现一个自定义的layer类，你可以重写绘制方法来做任何绘制。一个layer对象产生自定义的内容并不常见，但是它当然可以做到。比如，`CATiledLayer `管理一张大图片，通过把它切成小块来分开绘制。由于只有layer有信息关于哪一块需要在何时绘制，它自己来管理绘制行为。

当继承时，你可以选用以下方法的一种来绘制layer的内容：

* 重载`display`方法，并且用它来给layer直接赋值`contents`属性。

* 重载`drawInContext:`方法来绘制到被提供的graphics context里。

至于用哪种方法，取决于你需要对绘制过程的控制有多强。`display`方法是更新layer内容的主要入口，所以重载这个方法你可以完全控制整个过程。同时意味着你需要创建`CGImageRef`对象并且赋值给`contents `属性。如果你只是想绘制（或者想让你的layer管理绘制操作），你可以重载`drawInContext:`方法，让layer来为你创建backing store。

### Tweaking the Content You Provide

当你赋值图片给layer的`contents`属性时，layer的`contentsGravity`属性决定了图片会怎样被放置在layer的`bounds`里。默认的，如果图片大于或小于当前的`bounds`，layer会缩放图片来放置到可用的区域。如果layer的`bounds`的aspect ratio和图片的aspect ratio（就是长宽比）不一样，可能造成图片被扭曲。可以修改`contentsGravity `来确保图片尽可能的展示正常。

可以赋值给`contentsGravity`的属性大致被分为两类:

* position-based，基于位置的，可以使图片对齐一角或者一边，或者直接放在中间。

* scaling-based，基于缩放的，可以根据aspect ratio来填充。

### Working with High-Resolution Images

layer没有任何关于设备屏幕分辨率的解决方法。layer只是存储一个指向bitmap的指针并且把它尽可能的最好的展示在所给的像素里。如果你赋值一个图片给layer的`contents`属性，你必须通过设置`contentsScale`属性来告诉CoreAnimation关于图片的分辨率。该属性默认是1.0，这是图片被显示在一般屏幕上的值。如果图片在retina屏上，需要设置成2.0。

改变`contentsScale`的值只在你直接给layer赋值bitmap的时候需要。layer-backed view会自动根据屏幕分辨率来提供适当的值。

## Adjusting a Layer’s Visual Style and Appearance

layer有内置的可视化装饰物，比如border和background color，可以用来补充layer的内容。由于这些可视化装饰物不要求你做任何绘制，折让layer成为单独的实体成为可能。你只需要设置一些属性，layer会处理绘制和动画。具体有哪些属性，可以参见[Layer Style Property Animations](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/LayerStyleProperties/LayerStyleProperties.html#//apple_ref/doc/uid/TP40004514-CH10-SW1)。

### Layers Have Their Own Background and Border

layer可以在本来的图片内容上显示被填充过的背景和描绘过的边框。background color在图片内容之下，而border在图片之上。如果layer有sublayers，它们也在border之下。由于背景色在图片之下，所以背景色会在图片透明的部分露出来。

如果background color是不透明的，可以考虑设置layer的`opaque`属性为`YES`。这样可以提高显示效率。但是如果layer有圆角，那么绝不能这样做。

## Layers Support a Corner Radius

你可以为layer创建圆角。圆角是一种遮挡部分layer区域的视觉装饰物，由于需要提交透明的遮罩，圆角并不会影响layer的`contents`，除非`masksToBounds`被设置为`YES`。但是，圆角一定会影响到背景色和边框的绘制。

You can create a rounded rectangle effect for your layer by adding a corner radius to it. A corner radius is a visual adornment that masks part of the corners of the layer’s bounds rectangle to allow the underlying content to show through, as shown in Figure 2-4. Because it involves applying a transparency mask, the corner radius does not affect the image in the layer’s contents property unless the masksToBounds property is set to YES. However, the corner radius always affects how the layer’s background color and border are drawn.

## 🤓Layers Support Built-In Shadows

CALayer类含有几个属性来配置阴影效果。阴影让layer看起来像是浮在它下面的内容之上。你可能会觉得有用，你可以改变阴影的颜色。

阴影的`opacity`默认为0，这样可以高效的隐藏阴影。把它改成任何非0值将会使CoreAnimation来绘制阴影。由于阴影在layer之下，你可能还需要修改阴影的偏移量来让你看到它。

给layer增加阴影的时候，阴影是layer内容的一部分，但是实际上它超出了layer的`bounds`。因此，如果你启用了`masksToBounds`，阴影就被边缘给切了。如果layer包含透明部分，这可能会造成一种奇怪的效果，阴影的一部分还可见，但是超出layer的部分不行了。如果你确实需要兼顾阴影和`masksToBounds`的效果，你可以用两个layer来实现。一个存放内容，包含mask，然后把它重合在另外一个有阴影效果的同样大小的layer上面。


## Adding Custom Properties to a Layer

CAAnimation和CALayer支持通过key-value coding自定义属性。你可以用这个方法给layer增加数据，然后用一个自定义的key把它取出来。你还可以关联actions在自定义属性上，那样的话，当你改变属性，一个对应的动画就会执行。

[Key-Value Coding Compliant Container Classes](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Key-ValueCodingExtensions/Key-ValueCodingExtensions.html#//apple_ref/doc/uid/TP40004514-CH12-SW3)。

[Changing a Layer’s Default Behavior](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/ReactingtoLayerChanges/ReactingtoLayerChanges.html#//apple_ref/doc/uid/TP40004514-CH7-SW1)，这篇后面会翻译到。













