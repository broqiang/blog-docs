+++
title = "对角线遍历"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2019-05-13T16:07:16
updated_at = 2019-05-13T16:07:16
description = "leetcode 498. 对角线遍历， 给定一个含有 M x N 个元素的矩阵（M 行，N 列），请以对角线遍历的顺序返回这个矩阵中的所有元素"
tags = ["go", "算法", "数组"]
+++

题目来自： [https://leetcode-cn.com/problems/diagonal-traverse/](https://leetcode-cn.com/problems/diagonal-traverse/)

## 原题

### 描述

给定一个含有 M x N 个元素的矩阵（M 行，N 列），请以对角线遍历的顺序返回这个矩阵中的所有元素，对角线遍历如下图所示。

### 示例

```bash
输入:
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]

输出:  [1,2,4,7,5,3,6,8,9]
```

解释：

![示例](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/diagonal_traverse.png)

### 说明

给定矩阵中的元素总数不会超过 100000 。

## 分析

这个题目的题意很容易理解，就是将每个坐标点的 nums[x][y] 对应的值放到一个数组中，先向右上移动，
当达到边界的时候再向左下移动，达到边界的之后再向右上移动 …… ，一直到所有的元素遍历一遍。

先假设下面几个变量，后面解释的时候容易理解一些：

+ row 一维数组的长度， 也就是对应图中的总行数

+ col 二维数组的长度， 也就是对应图中的总列数

+ x 一维数组的索引，也就是代表当前坐标是第几行

+ y 二维数组的索引，也就是代表当前坐标是第几列

### 向右上移动的时候会有下面几种情况：

+ 坐标在第 1 行（ x=0 ）时，例如中 1 的右上位置， 不能再向上移动， 因为上面已经倒了顶部，
再移动就越界了， 所以只能向右移动一步（y+1)， 并且下一次就要开始向左下移动了。

+ 坐标在最后一列，如图中 3 的位置，此时就不能再向右上移动了，要向下移动一步(x 不变，
y = col-1)， 并且下一次开始就要向左下移动了。

+ 剩下的情况， 就正常向右上移动 x--， y++ 即可。

### 向左下移动时候的几种情况：

这个其实和右上差不多，就是更换下 x， y 的坐标。

+ 坐标在第 1 列（ y=0 ）时，例如图中 4 的位置，这时不能再向左移动，需要向下移动一步，并且下
一次就要改变方向向右上移动了， y =0, x + 1 。

+ 坐标在最后一行， 如图中 8 的位置，此时就不能再向下移动（越界），要右移一步，并且下次移动转变
方向， y 不变， x = row -1 。

+ 剩下的就正常向左下移动 x+1, y-1 即可。

## 解法 1

上面说明如果看懂，这个就很简单了，也是我上来开始想到的思路。 就是按照顺序去 `++` `--` 坐标即可，
注意每种移动对应的两个越界点即可，详细见代码中的注释：

```go
func findDiagonalOrder(matrix [][]int) []int {
    // 矩阵的行和列
    row := len(matrix)
    if row == 0 {
        return nil
    }

    col := len(matrix[0])
    // 矩阵的横纵坐标，用来记录一维数组和二维数组对应的 key
    x, y := 0, 0
    // 标记是右上方向还是左下方向， 右上开始
    flag := true

    // 初始化记录结果的 slice， 容量是矩阵中坐标的数量
    res := make([]int, row*col)

    for i:=0; i < row*col; i++ {
        // 根据获得的 x， y 将数据添加
        res[i] = matrix[x][y]

        // 计算移动后的坐标，下次添加的时候使用

        // 右上移动 x-- , y++
        if flag {
            x--
            y++
            // 如果没有出界，就继续下一次
            if 0 <= x && y < col {
                continue
            }

            // 只要越界，就要开始相反的方向移动
            flag = false

            // 右上移动时，x < 0 ， y 没有越界，
            // 比如对应图中数字 1， 2 的上方位置， 此时的坐标是 -1, 1
            if x < 0 && y < col {
                x = 0
                continue
            }

            // 如果不是上面的越界方式，就是 x， y 都出界了，
            // 比如图中 3 右上的位置， 此时 x, y = -1, 3 ，因为第一排已经遍历完成，
            // 需要移动到下一行再开始，所以要 -1+2 , y 就是它的最大长度的坐标即可
            x = x+2
            y = col-1

            continue
        }

        // 如果是向左下移动
        x++
        y--

        // 如果没有越界，就继续下一次
        if 0 <= y && x < row {
            continue
        }

        // 只要有越界就该翻转方向了
        flag = true

        // 如果 y 越界， x 没有越界，就像图中 4 左下的位置，此时是 x, y = 2, -1
        if y < 0 && x < row {
            y = 0
            continue
        }

        // y 越界了， 就像图中 8 的左下，需要将坐标移动到 9 的位置
        y = y + 2
        x = row-1
    }

    return res
}
```

## 解法 2

这个和上一个思路是完全一致的，就是修改了下写法，提交后执行时间和内存消耗都有了些许提高。

```go
func findDiagonalOrder(matrix [][]int) []int {
    row := len(matrix)
    if row == 0 {
        return nil
    }

    col := len(matrix[0])
    x, y := 0, 0
    flag := true

    res := make([]int, row*col)

    for i:=0; i < row*col; i++ {
        res[i] = matrix[x][y]
        if flag {
            if y == col -1 {
                x++
                flag = false
            } else if x== 0 {
                y++
                flag = false
            } else {
                x--
                y++
            }
        } else {
            if x == row -1 {
                y++
                flag = true
            } else if y == 0 {
                x++
                flag = true
            } else {
                x++
                y--
            }
        }
    }

    return res
}
```

## 解法 3

观察对角线索引 x+y 的和，会发现，向右上移动的时候的 x+y 为偶数， 向左下移动的时候 x+y 为奇数，
其他的就和上面的解法 2 思路是一致的。

```go
func findDiagonalOrder(matrix [][]int) []int {
    row := len(matrix)
    if row == 0 {
        return nil
    }

    col := len(matrix[0])
    x, y := 0, 0

    res := make([]int, row*col)

    for i:=0; i < row*col; i++ {
        res[i] = matrix[x][y]

        // 偶数，向右上
        if (x+y) %2 == 0 {
            if y == col -1 {
                x++
            } else if x == 0 {
                y++
            } else {
                x--
                y++
            }
        } else {
            if x == row -1 {
                y++
            } else if y == 0 {
                x++
            } else {
                x++
                y--
            }
        }
    }

    return res
}
```
