---
title: "二叉树的遍历(Go)"
subtitle: ""
date: 2022-02-20T17:51:01+08:00
draft: false
# page title background image
bg_image: "images/backgrounds/page-title.jpg"
# about image
image: "images/blog/golang.jpg"
# meta description
description: "二叉树的遍历"
# categories
categories: ["go"]
# tags
tags: ["go", "二叉树", "算法"]
# type
type: "post"

aliases: ""
---

## 1.前序遍历

{{<callout note "注意">}}
前序遍历：中左右

压栈顺序：右左中
{{</callout>}}

```go
// 递归
func preorderTraversal(root *TreeNode) (res []int) {
    var traversal func(node *TreeNode)
    traversal = func(node *TreeNode) {
    if node == nil {
        return
    }
        res = append(res, node.Val)
        traversal(node.Left)
        traversal(node.Right)
    }
    traversal(root)
    return res
}

// 迭代
func preorderTraversal(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    var stack = list.New() //栈
    res := []int{}         //结果集
    stack.PushBack(root)
    var node *TreeNode
    for stack.Len() > 0 {
        e := stack.Back()
        stack.Remove(e)     //弹出元素
        if e.Value == nil { // 如果为空，则表明是需要处理中间节点
            e = stack.Back() //弹出元素（即中间节点）
            stack.Remove(e)  //删除中间节点
            node = e.Value.(*TreeNode)
            res = append(res, node.Val) //将中间节点加入到结果集中
            continue                    //继续弹出栈中下一个节点
        }
        node = e.Value.(*TreeNode)
        //压栈顺序：右左中
        if node.Right != nil {
            stack.PushBack(node.Right)
        }
        if node.Left != nil {
            stack.PushBack(node.Left)
        }
        stack.PushBack(node) //中间节点压栈后再压入nil作为中间节点的标志符
        stack.PushBack(nil)
    }
    return res
}
```

## 2.中序遍历

{{<callout note "注意">}}
中序遍历：左中右

压栈顺序：右中左
{{</callout>}}

```go
// 递归
func inorderTraversal(root *TreeNode) (res []int) {
    var traversal func(node *TreeNode)
    traversal = func(node *TreeNode) {
        if node == nil {
            return
        }
        traversal(node.Left)
        res = append(res, node.Val)
        traversal(node.Right)
    }
    traversal(root)
    return res
}

// 迭代
func inorderTraversal(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    stack := list.New() //栈
    res := []int{}      //结果集
    stack.PushBack(root)
    var node *TreeNode
    for stack.Len() > 0 {
        e := stack.Back()
        stack.Remove(e)
        if e.Value == nil { // 如果为空，则表明是需要处理中间节点
            e = stack.Back() //弹出元素（即中间节点）
            stack.Remove(e)  //删除中间节点
            node = e.Value.(*TreeNode)
            res = append(res, node.Val) //将中间节点加入到结果集中
            continue                    //继续弹出栈中下一个节点
        }
        node = e.Value.(*TreeNode)
        //压栈顺序：右中左
        if node.Right != nil {
            stack.PushBack(node.Right)
        }
        stack.PushBack(node) //中间节点压栈后再压入nil作为中间节点的标志符
        stack.PushBack(nil)
        if node.Left != nil {
            stack.PushBack(node.Left)
        }
    }
    return res
}
```

## 3.后序遍历

{{<callout note "注意">}}
后续遍历：左右中

压栈顺序：中右左
{{</callout>}}

```go
// 递归
func postorderTraversal(root *TreeNode) (res []int) {
    var traversal func(node *TreeNode)
    traversal = func(node *TreeNode) {
        if node == nil {
            return
        }
        traversal(node.Left)
        traversal(node.Right)
        res = append(res, node.Val)
    }
    traversal(root)
    return res
}

// 迭代
func postorderTraversal(root *TreeNode) []int {
    if root == nil {
        return nil
    }
    var stack = list.New() //栈
    res := []int{}         //结果集
    stack.PushBack(root)
    var node *TreeNode
    for stack.Len() > 0 {
        e := stack.Back()
        stack.Remove(e)
        if e.Value == nil { // 如果为空，则表明是需要处理中间节点
            e = stack.Back() //弹出元素（即中间节点）
            stack.Remove(e)  //删除中间节点
            node = e.Value.(*TreeNode)
            res = append(res, node.Val) //将中间节点加入到结果集中
            continue                    //继续弹出栈中下一个节点
        }
        node = e.Value.(*TreeNode)
        // 压栈顺序：中右左
        stack.PushBack(node) //中间节点压栈后再压入nil作为中间节点的标志符
        stack.PushBack(nil)
        if node.Right != nil {
            stack.PushBack(node.Right)
        }
        if node.Left != nil {
            stack.PushBack(node.Left)
        }
    }
    return res
}
```
