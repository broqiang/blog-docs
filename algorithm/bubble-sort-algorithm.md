+++
title = "冒泡排序算法"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2018-05-20T01:17:44
updated_at = 2018-05-20T01:17:44
description = ""
tags = ["go", "算法"]
+++

## 冒泡排序算法

冒泡排序（Bubble Sort），是一种计算机科学领域的较简单的排序算法。
它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。
这个算法的名字由来是因为越大的元素会经由交换慢慢“浮”到数列的顶端，故名“冒泡排序”。

算法介绍及原理见： [冒泡排序-百度百科](https://baike.baidu.com/item/%E5%86%92%E6%B3%A1%E6%8E%92%E5%BA%8F/4602306?fromtitle=%E5%86%92%E6%B3%A1%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95&fromid=11201694&fr=aladdin)

## Go 语言实现

```golang
package main

import "fmt"

func main() {
	number := []int{95, 45, 15, 78, 84, 51, 24, 12}
	n := BubbleSort(number)
	fmt.Println(n)
}

func BubbleSort(number []int) []int {
	length := len(number)

	for i := 0; i < length; i++ {
		for j := 0; j < length-1; j++ {
			if number[j] > number[j+1] {
				tmp := number[j+1]
				number[j+1] = number[j]
				number[j] = tmp
			}
		}
	}

	return number
}
```