+++
title = "翻转字符串里的单词"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2019-05-15T14:43:39
updated_at = 2019-05-15T14:43:39
description = "leetcode 151. 翻转字符串里的单词, go 的多种实现"
tags = ["go", "算法", "字符串"]
+++

题目来自： [151. 翻转字符串里的单词](https://leetcode-cn.com/problems/reverse-words-in-a-string/)

## 题目描述

给定一个字符串，逐个翻转字符串中的每个单词。

### 示例 1：

```bash
输入: "the sky is blue"
输出: "blue is sky the"
```

### 示例 2：

```bash
输入: "  hello world!  "
输出: "world! hello"
```

解释: 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。

### 示例 3：

```bash
输入: "a good   example"
输出: "example good a"
```

解释: 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。

## 说明

无空格字符构成一个单词。
输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。

进阶：

请选用 C 语言的用户尝试使用 O(1) 额外空间复杂度的原地解法。

## 分析

这个是一道比较简单的题目， 实现的方式也很多了， 直接开干了

### 解法 1 正则分割

这个是最简单的方式， 使用系统函数可以很简单的实现。

```go
func reverseWords(s string) string {
    // 先将字符串去除首尾空白， 然后通过空白符拆分成数组
    arr := regexp.MustCompile(`\s+`).Split(strings.TrimSpace(s), -1)

    // 将数组反转
    for i, j := 0, len(arr)-1; i < j; i, j = i+1, j-1 {
        arr[i], arr[j] = arr[j], arr[i]
    }

    // 将数组拼接成字符串返回
    return strings.Join(arr, " ")
}
```

### 解法 2 不用任何系统函数

这个有点啰嗦，有点暴力，性能也不太好

```go
func reverseWords(s string) string {
    // 不用内置函数实现， 有点啰嗦
    res := ""

    // 用来记录是否是第一组， 其实可以将它放在一个 slice 中， 最终拼接空格，就不需要这个了
    flag := true

    left, right := len(s)-1, len(s)

    for ; left >= 0; left -- {
        if s[left] == ' ' {
            if left+1 < right {
                if !flag {
                    res += " "
                }
                res += s[left+1:right]
                flag = false
            }

            right = left
        }
    }

    if left+1 < right {
        if !flag {
            res += " "
        }
        res += s[left+1:right]
    }


    return res
}
```

## 解法 3 使用 slice 保存结果

思路和 2 没什么区别， 只是换成用一个 slice 保存分割的值， 最后再合并结果

```go
func reverseWords(s string) string {
    // 不用内置函数实现，字符串拼接，速度还不如内置函数快（虽然用了正则）
    // 试试使用 slice 保存， 最后拼接字符串
    res := make([]string, 0)

    left, right := len(s)-1, len(s)

    for ; left >= 0; left -- {
        if s[left] == ' ' {
            if left+1 < right {
                res = append(res, s[left+1:right])
            }

            right = left
        }
    }

    if left+1 < right {
        res = append(res, s[left+1:right])
    }


    return strings.Join(res, " ")
}
```

经测试， 速度和消耗都比前两种要小了

## 解法 4 使用 buffer

思路和 2 、 3 一样，只是将分割结果保存到 buffer 中， 最后再返回。

```go
func reverseWords(s string) string {
	buf := bytes.Buffer{}

	left, right, flag := len(s)-1, len(s), true

	for ; left >= 0; left-- {
		if s[left] == ' ' {
			if left+1 < right {
				if !flag {
					buf.WriteByte(' ')
				}
				buf.WriteString(s[left+1 : right])
				flag = false
			}

			right = left
		}
	}

	if left+1 < right {
		if !flag {
			buf.WriteByte(' ')
		}
		buf.WriteString(s[left+1 : right])
		flag = false
	}

	return buf.String()
}
```
