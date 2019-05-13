+++
title = "杨辉三角"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2019-05-13T23:33:25
updated_at = 2019-05-13T23:33:25
description = "leetcode 118. 杨辉三角 Go 语言解法"
tags = ["go", "算法", "数组"]
+++

题目来自： [leetcode 118. 杨辉三角](https://leetcode-cn.com/problems/pascals-triangle)

## 题目描述

给定一个非负整数 numRows，生成杨辉三角的前 numRows 行。

![PascalTriangleAnimated2](https://i.loli.net/2019/05/13/5cd98f4223bb157639.gif)

> 在杨辉三角中，每个数是它左上方和右上方的数的和。

## 示例

```bash
输入: 5
输出:
[
     [1],
    [1,1],
   [1,2,1],
  [1,3,3,1],
 [1,4,6,4,1]
]
```

## 分析

百科对 [杨辉三角形](https://zh.wikipedia.org/wiki/%E6%9D%A8%E8%BE%89%E4%B8%89%E8%A7%92%E5%BD%A2)
的解释， 用直白一点公式就是：

```bash
triangle[i][j] = triangle[i-1][j-1] + triangle[i-1][j]
```

根据这个公式，就可以很容易的计算出结果， 注意下下标越界就可以了，因为 i-1 行会比 i 行少一列，
所以注意下左侧的 0 越界和右侧的 length 。

## 解法 1

这个是我第一时间想出来的写法， 不太优雅， 不过可以很容易表述出公式

```go
func generate(numRows int) [][]int {
    // 初始化返回的 slice
    res := make([][]int,numRows)

    // 如果传入的是 0 就返回个空的 slice ， 如果不反悔，下面手动添加第一行数据的时候就 painc
    if numRows == 0 {
        return res
    }

    // 手动添加第一行的数据
    res[0] = []int{1}

    for i := 1; i < numRows; i++ {
        // 创建二级数组， 长度永远比上一个长一个，因为 i 是索引， 所以 +1
        sub := make([]int, i+1)

        for j := 0; j < i+1; j++ {
            // 获取左上的值， 给一个默认的 0， 越界的时候它就是 0， 不越界去上一行去取值
            left := 0
            if j > 0 {
                left = res[i-1][j-1]
            }

            // 获取右上的值，给一个默认的 0， 越界的时候它就是 0， 不越界去上一行去取值
            right := 0
            if j < i {
                right = res[i-1][j]
            }

            sub[j] = left + right
        }

        // 直接将新获取到的节点添加
        res[i] = sub
    }

    return res
}
```

## 解法 2

这个和解法 1 没有本质区别， 只是代码稍微整齐了一点

```go
func generate(numRows int) [][]int {
    res := make([][]int,numRows)
    var val int

    for i := 0; i < numRows; i++ {
        if i == 0 {
            res[i] = []int{1}
            continue
        }

        sub := make([]int, i+1)
        for j := 0; j < i+1; j++ {
            if j == 0 || j == i {
                val = 1
            } else {
                val = res[i-1][j-1] + res[i-1][j]
            }
            sub[j] = val
        }

        res[i] = sub
    }

    return res
}
```