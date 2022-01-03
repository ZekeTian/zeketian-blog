---
layout: post
title: LeetCode 80-Remove Duplicates from Sorted Array II
categories: 算法
tags: LeetCode 数组 双指针
author: ZekeTian
---



* content
{: toc}


本文主要讲解 LeetCode 中第 80 号问题 [Remove Duplicates from Sorted Array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/) 的解法，使用的思路与第 26 号问题 [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) 一样，是 26 号问题思路的变种。



## 1. 题目描述

给定一个排好序的数组 nums，在原数组中同删除重复元素，从而使得数组中每个元素最多出现两次，并且返回数组新的长度。

**示例：**

给定数组 nums = [1,1,1,2,2,3]，函数返回长度 5，数组的前 5 个元素为：1, 1, 2, 2,3。不需要考虑 nums[5] 的值。

**要求：**

不要创建一个新的数组，你必须要以 O(1) 的空间复杂度来修改输入的原数组。不需要考虑新数组长度之后的元素。



## 2. 求解思路

本题是第 26 号问题 [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) 的变种，只不过在本题中，**元素最多可以出现两次**。求解思路实际上与 26 号问题一样，只需要将 26 号问题的求解代码改动一点即可。在看本题的思路之前，请先看 26 号问题的求解思路，详见本人的另一篇[博客](https://www.zeketian.com/2020/08/04/leetcode-26-remove-duplicates-from-sorted-array/)。

在 26 号问题的求解代码中，遍历数组 nums 时，确定 nums[i] 在数组的 [0,k) 之间未出现过的条件是 `nums[i] > nums[k-1]`。而在本题中，由于每个元素最多可以出现两次，所以当 nums[i] 能放进 nums[k] 处时，nums[i] 是可以与 nums[k-1] 相同的，但是与 nums 中 [0, k-2] 之间不能有相同的元素（即 **nums[i] 在 nums 中 [0, k-2] 之间未出现过**）。所以此时，判断条件应该为：`nums[i] > nums[k-2]` 。

实际上，本题中若要求**每个元素最多可以重复出现 r 次**，则判断条件只需要更改为：`nums[i] > nums[k-r]` 即可。

本题的具体代码如下，同时也已上传至 [GitHub](https://github.com/ZekeTian/LeetCode/blob/master/src/array/_80_RemoveDuplicatesFromSortedArray2.java) 中。

```java
class Solution {
    public int removeDuplicates(int[] nums) {
    	if (null == nums || 0 == nums.length) {
    		return 0;
    	}
    	
    	int r = 2; // 允许重复的次数
    	if (nums.length <= r) {
    		return nums.length;
    	}
    	
    	int k = r; // nums 中 [0, k) 之间，每个元素最多重复 r 次
    	
    	for (int i = r; i < nums.length; ++i) {
    		// 判断 nums[i] 是否可以放进去
    		// 因为每个元素可以重复 r 次，所以 nums[i] 可以与 nums[k-1] 相同(r>=2)，但是不能与 nums 的 [0,k-r] 之间的元素相同
    		// 为了确保 nums[i] 与 nums 的 [0,k-r] 之间的元素不相同，则需要保证 nums[i] > nums[k-r]。
    		// 因为数组有序，nums[i] > nums[k-r] 时，实际上 nums[i] 就会大于 nums 的 [0,k-r] 之间所有的元素
    		// 如果 r=1 ，则 nums[i] 也不能与 nums[k-1] 相同。此时，下面的代码同样适用，因为也可以通过  nums[i] > nums[k-r] 保证条件。
    		
    		if (nums[i] > nums[k-r]) {
    			nums[k] = nums[i];
    			++k; // 放进了 nums[i] 后， k 向后移动
    		}
    	}
    	
    	return k;
    }
}
```



## 3. 总结

本题只是第 26 号问题的一个变种，求解思路也是相同的，只需要简单地更改一个判断条件即可。