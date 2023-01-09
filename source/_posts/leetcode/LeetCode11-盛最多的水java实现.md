---
title: LeetCode11.盛最多的水java实现
abbrlink: b723289f
date: 2019-02-23 23:50:56
tags: leetcode
categories: 算法
---
# 题目
给定 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0)。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
<!-- more -->
说明：你不能倾斜容器，且 n 的值至少为 2。

![](https://user-gold-cdn.xitu.io/2020/2/28/170877cf8a513b62?w=1035&h=500&f=png&s=52934)

图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。

 

示例:

输入: [1,8,6,2,5,4,8,3,7]
输出: 49
# 思路
这道题要找到最小的面积，可以使用两个for循环来实现，但那样时间复杂度会变为O(n*2)，为了缩短运行时间，可以从两边往中间缩小，高度低的下标往中间缩，如果面积比原来大，则替换为最大面积

# 代码
class Solution {
    public int maxArea(int[] height) {
        int low=0; // 容器左部
        int high=height.length-1; // 容器右部
        int h,area=0; // 声明容器高度，初始化面积
        do{
            if(height[low]>height[high])
                h=height[high];
            else
                h=height[low];
            if(area<h*(high-low))
                area=h*(high-low);
            if(height[low]>height[high]) // 判断那边刻度小，移动下标
                high--;
            else
                low++;
        }while(high-low>=1);
        return area;
    }
}