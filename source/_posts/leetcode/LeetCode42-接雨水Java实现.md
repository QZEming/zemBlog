---
title: LeetCode42.接雨水Java实现
abbrlink: 2044536d
date: 2019-03-29 22:15:45
tags: leetcode
categories: 算法
---
# 题目
给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

![](https://user-gold-cdn.xitu.io/2020/2/28/170877ef4e4d64a5?w=586&h=220&f=png&s=12913)


上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。 感谢 Marcos 贡献此图。

 

示例:

输入: [0,1,0,2,1,0,1,3,2,1,2,1]
输出: 6
# 思路
从两边找寻最大值，赋给maxLeft和maxRight，当height[left]>height[right]时left++，反之right--，如果遇到小于height[right]又小于maxLeft的值，则用maxLeft减去加到sum中，另一边也是同样的。

# 代码
```java
class Solution {
    public int trap(int[] height) {
        int sum=0;
        int maxLeft=0;
        int maxRight=0;
        int left=0;
        int right=height.length-1;
        while(left<right){
            if(height[left]<=height[right]){
                if(height[left]<maxLeft){
                    sum=sum+maxLeft-height[left];
                }else{
                    maxLeft=height[left];
                }
                left++;
            }else{
                if(height[right]<maxRight){
                    sum=sum+maxRight-height[right];
                }else{
                    maxRight=height[right];
                }
                right--;
            }
        }
        return sum;
    }
}
```