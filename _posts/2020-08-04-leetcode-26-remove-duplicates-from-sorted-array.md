---
layout: post
title: LeetCode 26-Remove Duplicates from Sorted Array
categories: 算法
tags: LeetCode 数组 双指针
author: ZekeTian
---


* content
{: toc} 


本文主要讲解 LeetCode 中第 26 号问题 [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) 的解法，使用同向双指针的思路。



## 1. 题目描述

给定一个排好序的数组 nums，在原数组中同删除重复元素，从而使得数组中每个元素只出现一次，并且返回数组新的长度。

**示例：**

给定数组 nums = [1,1,2]，函数返回长度 2，数组的前两个元素为：1,2。不需要考虑 nums[2] 的值。

**要求：**

不要创建一个新的数组，你必须要以 O(1) 的空间复杂度来修改输入的原数组。不需要考虑新数组长度之后的元素。



## 2. 求解思路

本题要求对数组去重，实际上也就是在数组中删除元素，所以思路与 27 号问题 [Remove Element](https://leetcode.com/problems/remove-element/) 的同向双指针思路类似。具体思路如下：

- 设置 i、k 两个指针，指针 i 用于遍历数组，指针 k 用于划分数组。在数组中，指针 k 的前面部分无重复元素，即在数组中 [0,k) 之间无重复元素。
- 遍历数组，判断当前元素是否在数组中 [0,k-1] 之间出现过。如果未出现过，则该元素是一个新元素，可以放进去

在本题中，关键问题是：**在遍历时，如何确定当前元素是否在数组的 [0,k-1] 之间出现过**。

要解决这个问题，需要充分利用给定数组有序的特点。

- 因为本题给定的数组是升序数组 nums（准确的说，是非降序），同时在遍历过程 i>=k，所以可以确定当前元素 nums[i] >= nums[k-1]。

- 又因为数组中 [0,k) 之间不能出现重复元素，因此 nums[i] 要想放进的话，则 nums[i] > nums[k-1]。

- 而当 nums[i] > nums[k-1] 时，即确定 nums[i] 大于 nums 中 [0,k-1] 之间所有的元素（因为 nums 中 [0,k-1] 之间的元素依然保持升序）。
- 从而可得出：当 nums[i] > nums[k-1] 时，nums[i] 不会与数组中 [0,k) 之间的元素相同，即**确定当前元素 nums[i] 在数组的 [0,k-1] 之间未出现过**。

本题的具体代码如下，同时也已上传至 [GitHub](https://github.com/ZekeTian/LeetCode/blob/master/src/array/_26_RemoveDuplicatesFromSortedArray.java) 中：

```java
class Solution {
	public int removeDuplicates(int[] nums) {
		if (null == nums || 0 == nums.length) {
			return 0;
		}

		int k = 1; // nums 中的 [0,k) 之间无重复元素
		
		for (int i = k; i < nums.length; ++i) {
			// 判断 nums[i] 是否可以放进去（即是否可以放在 nums[k] 处）
			// 当 nums[i] 与 nums 中 [0,k) 之间的元素全都不相同时，才能将 nums[i] 放进去
			// 因为数组有序（题目给定的是升序），则当 nums[i] > nums[k-1] 时，即可保证 nums[i] 大于 nums 中 [0,k) 之间所有的元素（即保证 nums[i] 不会与前面的重复）
			if (nums[i] > nums[k-1]) {
				nums[k] = nums[i];
				++k;
			}
		}
		
		return k;
	}
}
```



## 3. 总结

在求解本题的过程中，重点是要充分利用给定数组有序的特点，然后利用同向双指针的思路进行求解。