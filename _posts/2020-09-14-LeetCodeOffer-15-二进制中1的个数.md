---
layout:     post           # 使用的布局（不需要改）
title:      LeetCode-Offer15-二进制中1的个数
subtitle:   LeetCode-Offer15-二进制中1的个数 #副标题
date:       2020-09-14            # 时间
author:     甜果果                    # 作者
header-img: https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@master/assets/picgoimg/20200701171155.png  #背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - LeetCode
    - python
    - Offer

---

# [剑指 Offer 15. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/) [191. 位1的个数](https://leetcode-cn.com/problems/number-of-1-bits/)

tag: easy，位运算

**题目：**

请实现一个函数，输入一个整数，输出该数二进制表示中 1 的个数。例如，把 9 表示成二进制是 1001，有 2 位是 1。因此，如果输入 9，则该函数输出 2。

示例 1：

```
输入：00000000000000000000000000001011
输出：3
解释：输入的二进制串 00000000000000000000000000001011 中，共有三位为 '1'。
```

示例 2：

```
输入：00000000000000000000000010000000
输出：1
解释：输入的二进制串 00000000000000000000000010000000 中，共有一位为 '1'。
```

示例3：

```
输入：11111111111111111111111111111101
输出：31
解释：输入的二进制串 11111111111111111111111111111101 中，共有 31 位为 '1'。
```

# 方法一：逐位判断

-   根据 与运算 定义，设二进制数字 nn ，则有：
    -   若 n&1=0 ，则 n 二进制 最右一位 为 0 ；
    -   若 n&1=1 ，则 nn 二进制 最右一位 为 11 。
-   根据以上特点，考虑以下 循环判断 ：
    -   判断 n 最右一位是否为 1 ，根据结果计数。
    -   将 n 右移一位（本题要求把数字 n 看作无符号数，因此使用 无符号右移 操作）。
-   算法流程：
    -   初始化数量统计变量 res=0 。
    -   循环逐位判断： 当n=0 时跳出。
        -   res += n & 1 ： 若n&1=1 ，则统计数res 加一。
        -   n >>= 1 ： 将二进制数字 n 无符号右移一位（ Java 中无符号右移为 ">>>" ） 。
    -   返回统计数量 res。

-   复杂度分析：
    -   时间复杂度 O(log_2 n)： 此算法循环内部仅有 移位、与、加 等基本运算，占用 O(1)O(1) ；逐位判断需循环 log_2 n次，其中 log_2 n n 代表数字 n 最高位 1 的所在位数（例如 log_2 4 = 2, log_2 16 = 4。
    -   空间复杂度 O(1)O(1) ： 变量 resres 使用常数大小额外空间。

```python
class Solution:
    def hammingWeight(self, n):
        res = 0 
        while n:
            res += n & 1
            n >>= 1
        return res
```

>执行用时：40 ms, 在所有 Python3 提交中击败了77.44%的用户
>
>内存消耗：13.6 MB, 在所有 Python3 提交中击败了73.51%的用户