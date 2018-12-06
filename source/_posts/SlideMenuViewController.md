---
title: SlideMenuViewController
date: 2018-11-27 16:40:09
tags:
	- iOS开发 
	- custom UI
---

# SlideMenuViewController

非常常见的菜单选择，新的设计思路，使用UICollectionView，配合RxSwift代码，精简。

![.](SlideMenuViewController.gif)

## 通过两份协议，设置页面和更新页面:

```swift
protocol SlideMenuViewControllerDataSource: NSObjectProtocol {
    func slideMenuViewController(_ slideMenuViewController: UIViewController, viewControllerForItemAt index: Int) -> UIViewController
}

protocol SlideMenuViewControllerDelegate: NSObjectProtocol {
    func slideMenuViewController(_ slideMenuViewController: UIViewController, didChange viewController: UIViewController)
}
```

* **slideMenuViewController(:, viewControllerForItemAt:)** 这个方法可以根据index去决定需要显示的view controller，并且该index对应的view controller会被持有。只在该index第一次被选中时会触发。

* **slideMenuViewController(:, didChange:)** 这个方法在每次选中不同index都会回调，方便更新之前创建的view controller。

🤔具体的使用方法，可以参照代码。

## sample地址

### [10000ui](https://github.com/blurryssky/10000ui)
