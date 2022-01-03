---
layout: post
title: LeetCode 203-Remove Linked List Elements
categories: 算法
tags: LeetCode 链表 递归
author: ZekeTian
---





* content
{:toc}
本文主要讲解 LeetCode 中第 203 号问题 [Remove Linked List Elements](https://leetcode.com/problems/remove-linked-list-elements/) 的解法，提供循环求解和递归求解两种思路。



## 1. 题目描述

给定一个整数类型的链表和一个值，在链表中删除所有与指定值相等的元素。

**示例：**

```
Input:  1->2->6->3->4->5->6, val = 6
Output: 1->2->3->4->5
```



## 2. 求解思路

本题可以使用**循环删除**和**递归删除**两种思路，并重点介绍递归删除这种方式，所有代码均已经上传至 [GitHub](https://github.com/ZekeTian/LeetCode/blob/master/src/list/_203_RemoveLinkedListElements.java) 中。

### 2.1 循环删除

第一种思路，从头到尾遍历一遍链表，逐个删除与指定值相等的元素。此种思路较简单，只需要注意删除链表元素时的执行顺序，具体代码如下：

```java
class Solution1 {
    public ListNode removeElements(ListNode head, int val) {
        ListNode pre;
        ListNode cur;

        cur = head;
        // 删除链表中头部与 val 相等的部分
        while (cur != null && cur.val == val) {
            head = cur.next;
            cur.next = null;
            cur = head;
        }
        pre = head;

        // 删除链表中间及尾部与 val 相等的部分
        while (cur != null) {
            if (cur.val == val) {
                pre.next = cur.next;
                cur.next = null;
                cur = pre.next;
            } else {
                pre = cur;
                cur = cur.next;
            }
        }

        return head;
    }
}
```



### 2.2 递归删除

链表 `1->2->6->3->4->5->6->NULL`，实际上可以看成是节点 `1` 与子链表 `2->6->3->4->5->6->NULL` 组合形成，而链表 `2->6->3->4->5->6->NULL` 又可以看成是节点 `2` 与子链表 `6->3->4->5->6->NULL` 组合形成。因此，链表的结构满足递归的形式，可以采用递归求解。

如在链表`1->2->6->3->4->5->6->NULL` 中删除 `6` 的问题，可以进行如下一系列的转换：

- **S1:** 链表`1->2->6->3->4->5->6->NULL` 中删除 `6` ，转换成在链表 `2->6->3->4->5->6->NULL` 中删除 `6`，然后再连接上节点 `1`
- **S2:** 链表 `2->6->3->4->5->6->NULL` 中删除 `6`，转换成在链表 `6->3->4->5->6->NULL` 中删除 `6`，然后再连接上节点`2`
- **S3:** 链表 `6->3->4->5->6->NULL` 中删除 `6`，转换成在链表 `3->4->5->6->NULL` 中删除 `6`，然后再与 **S2**  步骤进行连接。（**在此步中，不能与 `6` 再进行连接，因为 `6` 是待删除的，所以只能与前一步 S2 进行连接**）
- **S4:** 链表 `3->4->5->6->NULL` 中删除 `6`，转换成在链表 `4->5->6->NULL` 中删除 `6`，然后再连接节点 `3`
- **S5:** 链表 `4->5->6->NULL` 中删除 `6`，转换成在链表`5->6->NULL`中删除 `6`，然后再连接节点 `4`
- **S6:** 链表`5->6->NULL`中删除 `6`，转换成在链表 `6->NULL`中删除 `6`，然后再连接节点 `5`
- **S7:** 在链表 `6->NULL` 中删除 `6`，转换成在链表 `NULL` 中删除 `6`，然后再与 **S6** 步骤进行连接。
- **S8:**  在链表 `NULL` 中删除 `6`，返回 `NULL`，直接结束

在上面的过程中，先一层一层递归深入进去，不断缩短链表的长度，最终使得链表长度为 0，从而达到递归的终止条件。之后，再一层一层递归往回走，将结果返回。**在返回的过程中，需要判断节点是否需要删除。如果需要删除，则该节点不会与链表进行连接。**

具体代码如下：

```java
class Solution2 {

    public ListNode removeElements(ListNode head, int val) {
        if (null == head) {
            return head;
        }

        // 删除链表中 [head.next, null) 范围内的指定值元素，然后返回删除元素后的链表。
        // 不断缩短链表的长短，直到链表中无元素，即满足递归结束的条件null == head
        ListNode ret = removeElements(head.next, val);

        if (head.val == val) {
            head.next = null;
            return ret; // 当 head 与待删除值相等时，则 head 对应的元素不需要添加到链表中，因此直接返回“删除元素后的链表”（即 ret）即可
        } else {
            head.next = ret; // 当 head 与待删除值不相等时，则 head 需要添加到链表中，因此需要将 head.next 与“删除元素后的链表”（即 ret） 连接起来
            return head;
        }
    }
}
```



## 3. 总结

链表虽然是一种线性结构，可以用常规的循环遍历方式处理。但是，实际上由于其结构也符合递归的形式，所以也可以采取递归方式处理。而递归方式的核心思想，是不断地缩短链表长度，然后处理。

此外，比较**循环**和**递归**两种方式实现的代码，可以看出**递归**方式更简洁。但是，只有理解了递归思想之后，才能真正明白代码的含义。

