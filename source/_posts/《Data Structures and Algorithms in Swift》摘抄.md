---
title: 《Data Structures and Algorithms in Swift》摘抄
date: 2019-02-19 16:28:23
tags: iOS开发
---


# [《Data Structures and Algorithms in Swift》书籍地址](https://store.raywenderlich.com/products/data-structures-and-algorithms-in-swift)

[raywenderlich](https://www.raywenderlich.com)是非常棒的辅导教程网站，上面发布了大量的iOS书籍，我也基本都看完了，获益匪浅。他们在传授知识的时候，也会细心告诉你合理的设计和使用方法，总之非常的棒。

这本书是关于数据结构与算法的，因为想要经常复习这些算法，所以我在这里做一些记录，希望通过简单的图形或文字回忆起来具体的逻辑。

《Data Structures and Algorithms in Swift》电子书需要的话可以给我留言或者发邮件

## linked list

![linked_list](linked_list.png)

insert | remove | search
--- | --- | ---
O(1) | O(1) | O(n)

**copy-on-write**

可以避免**cow**的情况:

1. 引用技术为1
```swift
guard !isKnownUniquelyReferenced(&head) else {
  return
}
```

2. 只插入头部时
![](linked_list_head_insertion.png)


## stacks

**LIFO**

## queues

![](queue.png)

**FIFO**

4种实现方式:

1. Array
2. DoublyLinkedList
3. RingBuffer
4. DoubleStack

## general purpose tree

![](tree.png)

2种遍历方式:

1. 深度优先
```swift
public func forEachDepthFirst(visit: (TreeNode) -> Void) {
  visit(self)
  children.forEach {
    $0.forEachDepthFirst(visit: visit)
  }
}
```

2. 宽度优先
```swift
public func forEachLevelOrder(visit: (TreeNode) -> Void) {
  visit(self)
  var queue = Queue<TreeNode>()
  children.forEach { queue.enqueue($0) }
  while let node = queue.dequeue() {
    visit(node)
    node.children.forEach { queue.enqueue($0) }
  }
}
```

## binary tree

![](binary_tree.png)

3种遍历方式:

1. 中序遍历 in-order
```swift
public func traverseInOrder(visit: (Element) -> Void) {
  leftChild?.traverseInOrder(visit: visit)
  visit(value)
  rightChild?.traverseInOrder(visit: visit)
}
```

2. 前序遍历 pre-order
```swift
public func traversePreOrder(visit: (Element) -> Void) {
  visit(value)
  leftChild?.traversePreOrder(visit: visit)
  rightChild?.traversePreOrder(visit: visit)
}
```

3. 后序遍历 post-order
```swift
public func traversePostOrder(visit: (Element) -> Void) {
  leftChild?.traversePostOrder(visit: visit)
  rightChild?.traversePostOrder(visit: visit)
  visit(value)
}
```

**遍历方式：in-order、pre-order**

## binary search tree

![](binary_search_tree.png)

满足以下条件的**binary tree**:

1. 左子树的值都小于父节点的值
2. 右子树的值都大于等于父节点的值

insert | remove | search
--- | --- | ---
O(1) | O(1) | O(logn)

## avl tree

![](avl.png)

是**binary search tree**

```swift
private func balanced(_ node: AVLNode<Element>) -> AVLNode<Element> {
  switch node.balanceFactor {
  case 2:
    if let leftChild = node.leftChild, leftChild.balanceFactor == -1 {
      return leftRightRotate(node)
    } else {
      return rightRotate(node)
    }
  case -2:
    if let rightChild = node.rightChild, rightChild.balanceFactor == 1 {
      return rightLeftRotate(node)
    } else {
      return leftRotate(node)
    }
  default:
    return node
  }
}
```

## tries

![](tries.png)

**.**标识是否为中止单词

## binary search

![](binary_search.png)

Require 2 conditions:

1. Conform protocol `RandomAccessCollection`.
2. Be sorted.

## heaps

![](heap.png)

是完整二叉树，一层满了才能去下层

siftDown, siftUp

insert | remove | search
--- | --- | ---
O(logn) | O(logn)| O(n)

## priority queue


1. Max-priority, where the element at the front is always the largest.
2. Min-priority, where the element at the front is always the smallest.

实现方式:

1. Sorted array.
2. Avl tree.
3. Heap.

3.Heap is the best choice.

## O(n2) sorting


1. Bubble
2. Selection. Find the lowest value and swap to i.
3. Insertion. 和冒泡算法不同的是，被交换的值会一直被比较到最后的位置。

## merge sort

不停的分割直到只有1个值，然后把分割后的值两两合并，合并的时候排序。
O(logn).


## radix sort

不通过比较，按位数依次放进“篮子”里，最后再拍平成数组。
O(k*n).k is the longest digit.

## heap sort

![](heap_sort.png)

swap and siftDown. O(logn).

## quicksort

![](quick_sort.png)

pivot的选择对时间复杂度影响很大，有一种策略是使用首位、中间位、末位三个数中的中间值作为pivot。
O(logn).

## graphs

![](graph.png)

![](graph_analysis.png)

## breadth first

![](breadth_first.png)

## depth first

![](depth_first.png)

## dijkstra shortest path

![](dijkstra.png)

## prims spanning tree

![](prims.png)