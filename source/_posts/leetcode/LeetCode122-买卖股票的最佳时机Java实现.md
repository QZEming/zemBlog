---
title: LeetCode122.买卖股票的最佳时机Java实现
abbrlink: 2424c68d
date: 2019-03-26 16:54:29
tags: leetcode
categories: 算法
---
# 题目
给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

如果你最多只允许完成一笔交易（即买入和卖出一支股票），设计一个算法来计算你所能获取的最大利润。

注意你不能在买入股票前卖出股票。

示例 1:

> 输入: [7,1,5,3,6,4]
> 输出: 5
> 解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
>    注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
示例 2:

> 输入: [7,6,4,3,1]
> 输出: 0
> 解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
# 思路
初始化买入值为第一天的股票价，收益值为0，从数组第二位开始，若小于买入值，则当前值为买入值，若当前值减去买入值大于当前收益值，则收益值变为当前值减去买入值。当只有一天时，收益值为0。

# 代码
```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices.length<2) // 特殊情况处理
            return 0;
        int buy=prices[0];
        int out=0;
        for(int index=1;index<prices.length;index++){
            if(prices[index]<buy){
                buy=prices[index];
            }else if(prices[index]-buy>out){
                out=prices[index]-buy;
            }
        }
        return out;
    }
}
```