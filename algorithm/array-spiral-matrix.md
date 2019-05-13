+++
title = "螺旋矩阵"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2019-05-13T18:26:20
updated_at = 2019-05-13T18:26:20
description = "leetcode 54. 螺旋矩阵"
tags = ["go", "算法", "数组"]
+++

来自： [leetcode 54. 螺旋矩阵](https://leetcode-cn.com/problems/spiral-matrix/)

## 原题：

### 题目描述

给定一个包含 m x n 个元素的矩阵（m 行, n 列），请按照顺时针螺旋顺序，返回矩阵中的所有元素。

### 示例 1

```bash
输入:
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]
输出: [1,2,3,6,9,8,7,4,5]
```

### 示例 2

```bash
输入:
[
  [1, 2, 3, 4],
  [5, 6, 7, 8],
  [9,10,11,12]
]
输出: [1,2,3,4,8,12,11,10,9,5,6,7]
```

## 题目分析

这个题目很容易理解， 就是从左上角开始沿着四周旋转，一直转到中心， 将获取的值保存， 就像是一个
向中心旋转的旋涡。

## 解法 1

暴力解法， 这个也是我第一时间想到的解法， 就是按照它的顺序去获取数据， 包括 4 个步骤：

+ 左向右（默认的开始方向）, x, y = 0, 0 开始， y++ 一直到 y = 右边界，此时顶部的一排已经
处理完了，给顶部标记（top) +1, 以后就不会再处理了， 并且行的坐标 x + 1， 并且记录方向的标记，
表示下次再来就要向下寻找。

+ 上向下， x++ 一直向下寻找，一直到 x = 下边界， 此时最右边一列已经处理完，将右边界
（right）- 1， y--（向左移动一列）， 标记下次要右向左寻找。

+ 右向左， x-- 一直向左寻找，一直到 x = 左边界（left）， 此时下面一行已经处理完了，
给下边界（bottom） - 1， x--（向上移动一行） ， 并标记下次要向上寻找。

+ 下向上, y-- 一直向上寻找， 一直到 y == 上边界（top）， 此时左侧一列已经处理完了， 给左边界
（left）+1， y++（向右移动一列）， 并标记下次要向右寻找。

就这样，按照顺序， 一直到找完全部元素为止。

```go
func spiralOrder(matrix [][]int) []int {
    row := len(matrix)
    if row == 0 {
        return nil
    }

    col := len(matrix[0])

    // 记录上，下，左， 右 的坐标
    top, bottom, left, right := 0, row-1, 0, col-1

    x, y := 0, 0
    // 记录的运行方向， 0 - 右， 1 - 下， 2 - 左， 3 - 上
    action := 0

    res := make([]int, row*col)

    for i :=0; i < row*col; i++ {
        res[i] = matrix[x][y]
        switch action {
        case 0: // 向右
            if y == right {
                top++
                x = top
                action = 1
            } else {
                y++
            }
        case 1: // 向下
            if x == bottom {
                right--
                y = right
                action = 2
            } else {
                x++
            }
        case 2: // 向左
            if y == left {
                bottom--
                x=bottom
                action=3
            } else {
                y--
            }
         case 3: // 向上
            if x == top {
                left++
                y = left
                action = 0
            } else {
                x--
            }
        }
    }

    return res

}
```

个人觉得这样不太好，应该不是题目想要的，执行完竟然还超越了 100% ， 意外。

## 解法 2

这个是在我写完上面的写法，去网上找到的一个 Java 的写法，改成 Go 之后的结果，原理和我写的那个
差不多， 就是写法不太一样，执行完也是 100% ， 消耗差不多。

```go
func spiralOrder(matrix [][]int) []int {
    row := len(matrix)
    if row == 0 {
        return nil
    }

    col := len(matrix[0])

    // 元素总数
    total := row*col

    // 上下左右 4 条边界
    top, bottom, left, right := 0, row-1, 0, col-1

    // 用来保存结果的 slice
    res := make([]int, 0, total)

    for {
        // 向右
        for i := left; i <= right; i++ {
            res = append(res, matrix[top][i])
        }
        top++
        if left > right || top > bottom {
            break
        }

        // 向下
        for i := top ; i <= bottom; i++ {
            res = append(res, matrix[i][right])
        }
        right--
        if left > right || top > bottom {
            break
        }

        // 向左
        for i := right;  left <= i; i-- {
            res = append(res, matrix[bottom][i])
        }
        bottom--
        if left > right || top > bottom {
            break
        }

        // 向上
        for i := bottom; top <= i; i-- {
            res = append(res, matrix[i][left])
        }
        left++
        if left > right || top > bottom {
            break
        }
    }

    return res

}
```

## 解法 3

没有解法 3 ， 网上看了一些，基本都是类似的，所以就没记录，但是我觉得上面两种都不是很好，希望看到
这篇文章的人如果有更好的解法可以添加，点击右上角的修改原文即可。