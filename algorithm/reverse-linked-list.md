+++
title = "反转链表"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2019-06-21T18:18:24
updated_at = 2019-06-21T18:18:24
description = ""
tags = ["go", "算法"]
+++

这是 leetcode 中的 206 题， 这里记录的是我的分析和解法， 欢迎有更好的思路和解法，
直接点击上面的修改原文。

## 题目描述

```bash
反转一个单链表。

示例:

输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
进阶:
你可以迭代或递归地反转链表。你能否用两种方法解决这道题？

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/reverse-linked-list
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

## 方法 1 迭代

遍历链表， 将当前节点的 next 指针改为指向上一个节点， 所以在改变 next 之前要将原本的 next
记录到一个变量中， 否则就找不到原本的下一个节点了。然后用当前节点替换上一个节点，再用前面记录的
临时变量（下一个节点）替换当前节点， 作为下轮遍历使用的数据。

c 代码：

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */

// 方法一， 迭代
struct ListNode* reverseList(struct ListNode* head){
    // 定义个上一个节点， 并且初始化为 NULL， 这样第一个节点的 next 就会变成 NULL
    struct ListNode* prev = NULL;
    // 定义当前节点， 初始就是 head
    struct ListNode* curr = head;
    // 一定一个临时保存当前节点数据的节点
    struct ListNode* tempNext;

    while (curr != NULL) {
        // 现将当前节点的下一个节点保存到临时变量中，
        // 因为下面修改了当前节点的 next 之后就会找不到 next 了
        tempNext = curr->next;
        // 将当前节点指向前一个节点， 这样就反转过来了
        curr->next = prev;
        // 将上一个节点指向当前节点，
        // 下一次遍历的时候就是包含了当前节点的所有的指向上一个节点的链表
        prev = curr;
        // 将当前节点替换成下一个节点， 进行下一轮遍历
        curr = tempNext;
    }

    return prev;
}
```

go 代码：

思路是一样的， 代码写起来舒服一些。

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
    // 方法 1 ， 迭代
    var prev *ListNode = nil

    for head != nil {
        next := head.Next
        head.Next = prev
        prev = head
        head = next
    }

    return prev
}
```

go 优化版：

和上面的思路一样， 利用了下 go 的特性， 节省了个临时变量， 执行后（leetcode中）时间没变，
内存更加节省了。

```go
func reverseList(head *ListNode) *ListNode {
    // 方法 1 ， 迭代， 改进版

    if head == nil {
        return nil
    }

    var prev, curr *ListNode = nil, head

    // 这里必须验证 curr.Next， 如果验证 curr 的话， .Next 如果是 nil
    // 就会出错了
    for curr.Next != nil {
        curr.Next, curr, prev = prev, curr.Next, curr
    }

    // 因为上面验证的是 curr.Next， 所以最后一个节点不会处理
    // 将最后一个节点链上之前所有处理完的节点
    curr.Next = prev

    return curr
}
```

## 方法2 递归 1

递归有点不是很容易想明白， 其实就是先处理最后一个节点的反转，然后从后向前反转，
核心就是将下一个节点的下一个节点，改成当前节点。

有点绕， 假设有三个节点， 1->2->3 ，原本 1 的下一个节点（2）的下一个节点是 3，
将 1 的下一个节点（2）的下一个节点指向当前节点（1）， 就完成了反转。

c 代码

```c
struct ListNode* reverseList(struct ListNode* head){
    // 方法 2， 递归 1
    if (!head || !head->next) {
        return head;
    }

    struct ListNode* p = reverseList(head->next);
    head->next->next = head;
    head->next = NULL;

    return p;
}
```

go 代码

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
    // 方法 2 ， 递归
    if head == nil || head.Next == nil {
        return head
    }

    p := reverseList(head.Next)
    head.Next.Next = head
    head.Next = nil

    return p

}
```

## 方法2 递归 2

其实和递归 1 的思路完全是一样的， 只是少处理一次给 next 赋空值的步骤，通过自定义新的函数，
每次传入上一个节点和当前节点， 更容易理解一些。 c 的执行完和上面的方法没上面区别， 不过 go
通过这个方式， 执行时间减少了一半（leetcode 结果中的时间）。

```c
struct ListNode* reverse(struct ListNode* prev, struct ListNode* curr) {
    if (!curr) {
        return prev;
    }

    struct ListNode* head = reverse(curr, curr->next);
    curr->next = prev;

    return head;
}


struct ListNode* reverseList(struct ListNode* head){
    // 方法 2， 递归 2

    return reverse(NULL, head);
}
```

```go
func reverseList(head *ListNode) *ListNode {
    // 方法 2 ， 递归 2

    return reverse(nil, head)
}

func reverse(prev, curr *ListNode) *ListNode {
    if curr == nil {
        return prev
    }

    p := reverse(curr, curr.Next)

    curr.Next = prev

    return p
}
```

本文来自 [字符串反转](https://broqiang.com/posts/reverse-linked-list)
随意转载，带上这个尾巴就可以了。
