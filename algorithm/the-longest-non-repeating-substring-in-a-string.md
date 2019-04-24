+++
title = "字符串中最长不重复子串"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2018-05-03T01:26:37
updated_at = 2018-05-03T01:26:37
description = "计算字符串中最长不重复子串，分别用 Go 和 PHP 实现"
tags = ["go", "PHP", "算法"]
+++

## 题目

从一个字符串中找到一个连续子串，该子串中任何两个字符不能相同，求子串的最大长度并输出一条最长不重复子串

### 思路

使用一个数组（PHP）或 map（Go） 记录到当前下标i的元素为止，最长的不重复子串长度。使用 start 记录上一个最长不重复子串的开始位置，使用 maxLength 记录子串的最大长度。

如 `abcabbc` 中, `a` 在第一次出现的时候是下标 0， 第二次出现就是 3 的位置，此时前面的长度就是 3 了，再计算就是 3 位置后面的长度了，这个描述的有点不太清楚，有空再完善，先将下面两个示例记录下来，分别用的 Go 语言和 PHP 语言实现。

## Go 示例

Go 因为有 rune 类型，实现起来这个非常的轻松，因为 rune 相当于增强型的 byte，它是一个 int32 的值，可以完全兼容中文，非常的方便。

```golang
package main

import "fmt"

func longestNonRepeatingStringSubstring(s string) int {
    lastMap := make(map[rune]int)
    start := 0
    maxLength := 0

    for i, ru := range []rune(s) {
        if lastVal, ok := lastMap[ru]; ok && lastVal >= start {
            start = lastVal + 1
        }
        if i-start+1 > maxLength {
            maxLength = i - start + 1
        }

        lastMap[ru] = i
    }
    return maxLength
}

func main() {
    fmt.Println(longestNonRepeatingStringSubstring("Hello World"))
}
```

## PHP 示例

```php
<?php
function lengthestNonRepeatingStringSubstring(string $str)
{
    // 将字符串拆分成数组
    $bytes = getBytes($str);

    $lastMap   = [];
    $start     = 0;
    $maxLength = 0;

    foreach ($bytes as $index => $byte) {
        if (isset($lastMap[$byte]) && $lastMap[$byte] > $start) {
            $start = $lastMap[$byte] + 1;
        }

        if ($index - $start + 1 > $maxLength) {
            $maxLength = $index - $start + 1;
        }

        $lastMap[$byte] = $index;
    }

    return $maxLength;
}

function getBytes(string $str)
{
    // 为了兼容中文，字符串分割不能使用 PHP 内置的 str_split，
    // 因为它分割的是 ASSIC 码，一个中文会被分割成 3 个 ASSIC 码，就变成乱码了
    return preg_split('/(?<!^)(?!$)/u', $str);
}

echo lengthestNonRepeatingStringSubstring("Hello World");
```

## 过程分析

暂时只记录下来的实现的代码，有空再写分析