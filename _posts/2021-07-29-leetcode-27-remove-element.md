---
layout: post
title: LeetCode 27-Remove Element
categories: 算法
tags: LeetCode 数组 双指针
author: ZekeTian
---



* content
{:toc}
本文主要讲解 LeetCode 中第 27 号问题 [Remove Element](https://leetcode.com/problems/remove-element/) 的思路，总共包含两种解法。第一种解法使用 283 号问题 [Move Zeroes](https://leetcode.com/problems/move-zeroes/) 的思路（同向双指针），第二种解法采用碰撞指针（针对此问题的另一种特殊情况）。



## 1. 题目描述

给定一个数组 nums 和一个值 val，在原数组中将所有给定值删除并且返回新的长度。

**示例 1：**

给定数组 nums = [3,2,2,3]， val = 3，函数返回长度为 2 ，数组的前两个元素为 2。

**示例 2：**

给定数组 nums = [0,1,2,2,3,0,4,2]， val = 2，函数返回长度为 5，数组的前 5 个元素应该为：0,1,3,0,4

**要求：**

不要创建一个新的数组，你必须要以 O(1) 的空间复杂度来修改输入的原数组。数组中元素的顺序可以发生改变，可以不用考虑新数组长度之后的元素。

## 2. 求解思路

本题中包含两种求解思路，一种是同向双指针，另一种是碰撞指针，所有代码均已经上传至 [GitHub](https://github.com/ZekeTian/LeetCode/blob/master/src/array/_27_RemoveElement.java) 中。

### 2.1 同向双指针

使用 283 号问题 [Move Zeroes](https://leetcode.com/problems/move-zeroes/) 的思路，将待删除值 val 移到数组的最后面（或者说是将非 val 的值移到数组的前面），然后返回新数组的长度，从而相当于在原始数组中将 val 删除。但是本题不同于 283 号问题，在本题中不需要考虑新数组长度之后的元素，所以最终不需要将 val 放到数组的最后面。

至于第 283 号问题的求解思路，可参考本人的另一篇[博客](https://www.zeketian.com/2020/07/28/leetcode-283-move-zeroes/)，而本题的具体代码如下：

```java
class Solution1 {
    public int removeElement(int[] nums, int val) {
    	int k = 0; // [0, k) 中不出现 val, 如果 nums 中有 val ，则 k 指针指向第一个 val 
    	
    	for (int i = 0; i < nums.length; ++i) {
    		if (val != nums[i]) {
    			nums[k] = nums[i];
    			++k;
    		}
    	}
    	
    	return k;
    }
}
```



### 2.2 碰撞双指针

当数组中待删除的元素较少时，Solution1 会进行大量的 nums[k] = nums[i] 操作。由于本题中不需要考虑数组中的元素顺序，因此可以采用碰撞指针的思路来减少 nums[k] = nums[i] 的操作次数。

**具体思路：**设置 i、k 两个指针，指针 i 向左移动，指针 k 向右移动寻找待删除值 val 

- 指针 k 向右移动，遇到非 val 时继续移到，直到找到 val
- 当指针 k 找到 val 时，指针 k 不移动，将 nums[i] 放到 nums[k] 处，i 向左移动

本题的具体代码如下：

```java
class Solution2 {
	public int removeElement(int[] nums, int val) {
		// [4,2,3,5,1], val = 4
		int k = 0; 
		int i = nums.length;
		while (k < i) {
			if (val == nums[k]) {
				// nums[k] 为 val 时，则将 nums[i-1] 放置到 nums[k] 处
				// 如果 nums[i-1] 也为 val ，则指针 k 保持不动，等待指针 i 找到一个非 val 值
				nums[k] = nums[i - 1];
				--i; // 此语句包含两种含义：（1）找到一个 val，则数组的长度减 1 （2）指针 i 向左移动
			} else {
				++k;
			}
		}
		
		return i;
	}
}
```

**注意：**代码中第 11 行代码 `--i;`  有两种含义，分别如下：

- 找到一个 val，则数组的长度减 1
- 指针 i 向左移动



### 3. 总结

本题的求解思路与 283 号问题类似，都可以采用同向双指针解决。当数组中待删除的元素较多时，这种同向双指针的思路较适合。如果当数组中待删除的元素较少时，使用碰撞双指针的思路较合适。

