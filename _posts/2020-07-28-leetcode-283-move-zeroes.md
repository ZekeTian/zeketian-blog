---
layout: post
title: LeetCode 283-Move Zeroes
categories: 算法
tags: LeetCode 数组 双指针
author: ZekeTian
---



* content
{: toc}
本文主要讲解 LeetCode 中第 283 号问题 [Move Zeroes](https://leetcode.com/problems/move-zeroes/) 的思路。从暴力求解开始，不断对方法进行优化，最终包含 4 种解决方法。




## 1. 题目描述

 给定一个数组，写一个函数将所有的 0 移到最后面，同时保持非 0 元素的相对顺序。

**示例：**

Input: [0,1,0,3,12]
Output: [1,3,12,0,0]

**要求：**

- 必须在原数组上操作，不能拷贝额外的数组
- 尽量减少操作次数



## 2. 求解思路

求解思路总共有 4 种，可以分为暴力求解、双指针求解两大类，所有求解代码均已上传至 [GitHub](https://github.com/ZekeTian/LeetCode/blob/master/src/array/_283_MoveZeroes.java) 中。

### 2.1 暴力求解

暴力求解的思路较简单，也是最容易想到，即利用一个新的数组依序存储非零元素，然后再这些非零元素再按序存回原数组中。**但是需要注意的是，本题中要求不能拷贝额外的数组，因此在求解本题时需要使用下面的双指针思路。**暴力求解的具体代码如下：

```java
class Solution1 {
	public void moveZeroes(int[] nums) {
		// 开辟一个新数组，存储非零元素
		ArrayList<Integer> noZero = new ArrayList<Integer>();
		for (int i = 0; i < nums.length; ++i) {
			if (0 != nums[i]) {
				noZero.add(nums[i]);
			}
		}
		
		// 将非 0 元素放在 nums 中
		for (int i = 0; i < noZero.size(); ++i) {
			nums[i] = noZero.get(i);
		}
		// 在 nums 的最后面放剩余的 0
		for (int i = noZero.size(); i < nums.length; ++i) {
			nums[i] = 0;
		}
    }
}
```



### 2.2 双指针（两数不交换）

本题中双指针思路：将数组分成两部分，前面部分放非零元素，后面部分放 0。假设从 k 处开始划分，则 **[0, k)** 处放非零 0 元素，**[k, num.length)** 放 0。具体过程如下：

- 放置 nums 中的非零元素

  遍历 nums 数组，将非零元素插入到前面非零部分（即 **[0, k)** 部分 ）的后面（nums[k] 处），然后 k 向后移动，直至遍历完 nums 中所有元素。

- 放置 nums 中的 0 

  遍历完数组后，k 处即为 0 开始放置的地方

在双指针思路中，指针 k 用于将数组划分为非零部分和零部分，指针 i 用于遍历数组，具体代码如下：

```java
class Solution2 {
	public void moveZeroes(int[] nums) {
		int k = 0; // [0, k) 区间内是非零元素
		for (int i = 0; i < nums.length; ++i) {
			if (0 != nums[i]) {
				nums[k] = nums[i]; // 遇到非 0 元素，则将其放在前面非 0 元素的后面，即 nums[k] 处
				++k; 
			}
		}
		
		// 后面从 k 处开始放 0 
		for (int i = k; i < nums.length; ++i) {
			nums[i] = 0;
		}
    }
}
```



### 2.3 双指针（两数交换）

在第 2 种方法中，遍历完 nums 数组后，依然还需要使用一个循环来放置 nums 中后面的 0 。而实际上，在遍历 nums 数组时可以将 0 与非零元素进行交换，从而可以避免后面的循环。因此在第 3 个方法中，采用两数交换的思路，具体代码如下：

```java
class Solution3 {
    public void moveZeroes(int[] nums) {
    	int k = 0;
    	for (int i = 0; i < nums.length; ++i) {
    		if (0 != nums[i]) { // 当 nums[i] = 0 时，k 不后移，此时 nums[k] = 0，并且 nums[k] 之后一直是 0
                // nums[k] 与 nums[i] 进行交换
    			int tmp = nums[k];
    			nums[k] = nums[i];
    			nums[i] = tmp;
    			++k;
    		}
    	}
    }
}
```

在第 5 行代码 `if (0 != nums[i]) ` 中，当 nums[i] = 0 时，k 不后移，此时 nums[k] = 0，并且 nums[k] 之后一直是 0。所以 nums[i] 与 nums[k] 进行交换，实际上就是后面 **[k, num.length)** 部分中将第一个非零元素 nums[i] 与 0（即 nums[k]）进行交换。

以题目中的数组 [0,1,0,3,12] 为例，交换过程如下：

- i = 0 时，k = 0 , nums[k]  = 0，不进行交换，数组为：[0,1,0,3,12]
- i = 1 时，k = 0，nums[k] = 0, 进行交换，数组为： [1,0,0,3,12]
- i = 2 时，k = 1，nums[k] = 0，不进行交换，数组为： [1,0,0,3,12]
- i = 3 时，k = 1，nums[k] = 0，进行交换，数组为： [1,3,0,0,12]
- i = 4 时，k = 2，nums[k] = 0，进行交换，数组为：[1,3,12,0,0]



### 2.4 双指针优化（两数交换）

在第 3 种方法中，如果 nums 数组中全部为非零元素，则 nums[i] 始终不为 0 ，k 与 i 同步自增，则在交换过程中实际上是两个相同的数在交换。针对此种情况，可以在交换之前，先判断 nums[i] 与 nums[k] 是否相等，如果不等再进行交换。并且若 nums[i] 与 nums[k] 不等，则说明 nums 数组含有 0，此时 nums[k] 必定为 0。因此根据这个特点，在交换过程中，可以将原来的三步操作缩减为两步。

简而言之，第 4  种方法主要存在如下的两点优化：

- 减少不必要的交换

- 将两个数交换过程的三步缩减为两步

  当  nums[i] 与 nums[k] 不相等时，nums[k] 为 0 并且之后始终为 0。所以将 nums[k] 与 nums[i] 进行交换时，将 nums[k] 赋值为 nums[i] 后可以直接将 nums[i] 赋值为 0，不需要中间变量 tmp

具体代码如下：

```java
class Solution4 {
	 public void moveZeroes(int[] nums) {
    	int k = 0;
    	for (int i = 0; i < nums.length; ++i) {
    		if (0 != nums[i]) {
    			// 当数组中全部为非零元素，则  nums[k] 不为 0，nums[k] 与 nums[i] 相等，则不需要交换；
    			// 当数组中含有 0 时，nums[k] = 0，此时才进行交换
    			if (nums[k] != nums[i]) {
    				nums[k] = nums[i];
        			nums[i] = 0;
    			}
    			++k;
    		}
    	}
	 }
}
```

## 3. 总结

本题是一道关于数组的问题，主要使用的方法是双指针。在使用双指针求解问题时，需要注意两个指针的各自含义。在本题中，指针 i 用于遍历数组，指针 k 用于对数组进行划分。