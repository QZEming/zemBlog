---
title: leetcode股票买卖问题js解法
tags: leetcode
categories: 算法
abbrlink: a9492bb0
date: 2020-02-29 19:35:50
---
# 714. 买卖股票的最佳时机含手续费
## 题目
给定一个整数数组 prices，其中第 i 个元素代表了第 i 天的股票价格 ；非负整数 fee 代表了交易股票的手续费用。
<!-- more -->
你可以无限次地完成交易，但是你每次交易都需要付手续费。如果你已经购买了一个股票，在卖出它之前你就不能再继续购买股票了。

返回获得利润的最大值。

示例 1:

输入: prices = [1, 3, 2, 8, 4, 9], fee = 2
输出: 8
解释: 能够达到的最大利润:  
在此处买入 prices[0] = 1
在此处卖出 prices[3] = 8
在此处买入 prices[4] = 4
在此处卖出 prices[5] = 9
总利润: ((8 - 1) - 2) + ((9 - 4) - 2) = 8.
注意:

0 < prices.length <= 50000.
0 < prices[i] < 50000.
0 <= fee < 50000.

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
## 思路
买卖股票的题目基本上都可以使用动态规划来完成，这里主要是要找出递归的方程

这里只有两种状态
1. 有股票在手，此时只能卖出股票
2. 无股票在手，此时只能买入股票

我们使用长度和prices一样的数组dp来存储每天对应的两个状态，每个成员都为一个长度为2的数组，分别存储两个状态对应现有的收益，这个收益可能是负数（在买入的时候）

然后通过递归方程
1. ```dp[i][0] = Math.max(dp[i-1][0],dp[i-1][1]+prices[i]-fee)```
2. ```dp[i][1] = Math.max(dp[i-1][1],dp[i-1][0]-prices[i])```

在最后一天手里没用股票就是最大值，也就是答案

## 代码实现
```javascript
var maxProfit = function(prices, fee) {
    let dp = Array(prices.length).fill([])
    dp[0] = [0,-prices[0]];
    for(let i=1;i<prices.length;i++){
        dp[i][0] = Math.max(dp[i-1][0],dp[i-1][1]+prices[i]-fee);
        dp[i][1] = Math.max(dp[i-1][1],dp[i-1][0]-prices[i]);
    }
    return dp[prices.length-1][0];
};
```
# 123. 买卖股票的最佳时机 III
## 题目
给定一个数组，它的第 i 个元素是一支给定的股票在第 i 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 两笔 交易。

注意: 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

示例 1:

输入: [3,3,5,0,0,3,1,4]
输出: 6
解释: 在第 4 天（股票价格 = 0）的时候买入，在第 6 天（股票价格 = 3）的时候卖出，这笔交易所能获得利润 = 3-0 = 3 。
     随后，在第 7 天（股票价格 = 1）的时候买入，在第 8 天 （股票价格 = 4）的时候卖出，这笔交易所能获得利润 = 4-1 = 3 。
示例 2:

输入: [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。   
     注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。   
     因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
示例 3:

输入: [7,6,4,3,1] 
输出: 0 
解释: 在这个情况下, 没有交易完成, 所以最大利润为 0。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
## 思路
这道题和上面一样，只是在这里限制了只能购买两次股票，所以我们这里设置五个状态，分别为
1. 没有买入过
2. 买入第一支股票
3. 卖出第一支股票
4. 买入第二支股票
5. 卖出第二支股票

递归方程为
1. ```dp[i][0] = 0```
2. ```dp[i][1] = Math.max(dp[i-1][1],dp[i-1][0]-prices[i])```
3. ```dp[i][2] = Math.max(dp[i-1][2],dp[i-1][1]+prices[i])```
4. ```dp[i][3] = Math.max(dp[i-1][3],dp[i-1][2]-prices[i])```
5. ```dp[i][4] = Math.max(dp[i-1][4],dp[i-1][3]+prices[i])```

最终的最大结果在没有买入过，卖出第一支股票，卖出第二支股票这三个值其中一个，只要取最大值就行
## 代码实现
```javascript
var maxProfit = function(prices) {
    let len = prices.length;
    if(len<2)
        return 0;
    let dp = Array(len);
    for(let i=0;i<len;i++){
        dp[i] = [0,0,0,0,0];
        dp[i][3] = -Infinity;
    }
    dp[0][1] = -prices[0];
    for(let i=1;i<len;i++){
        dp[i][0] = 0;
        dp[i][1] = Math.max(dp[i-1][1],dp[i-1][0]-prices[i]);
        dp[i][2] = Math.max(dp[i-1][2],dp[i-1][1]+prices[i]);
        dp[i][3] = Math.max(dp[i-1][3],dp[i-1][2]-prices[i]);
        dp[i][4] = Math.max(dp[i-1][4],dp[i-1][3]+prices[i]);
    }
    return Math.max(0,dp[len-1][2],dp[len-1][4])
};
```