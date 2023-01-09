---
title: LeetCode38.报数Java实现
abbrlink: e07e316a
date: 2019-02-23 00:55:56
tags: leetcode
categories: 算法
---
# 题目
报数序列是一个整数序列，按照其中的整数的顺序进行报数，得到下一个数。其前五项如下：
<!-- more -->
1.     1
2.     11
3.     21
4.     1211
5.     111221
1 被读作  "one 1"  ("一个一") , 即 11。
11 被读作 "two 1s" ("两个一"）, 即 21。
21 被读作 "one 2",  "one 1" （"一个二" ,  "一个一") , 即 1211。

给定一个正整数 n（1 ≤ n ≤ 30），输出报数序列的第 n 项。

注意：整数顺序将表示为一个字符串。

示例 1:

输入: 1
输出: "1"
示例 2:

输入: 4
输出: "1211"
# 思路
这里先确定第一个字符串为"1"，使用一个for循环，当循环到输入的数时停止并返回当前的字符串。在循环内部，每次先把字符串转为字符数组，将第一个字符标记出来用一个变量same存储，使用一个变量count存储重复的次数，如果下一个字符和same相同的话，就让count+1，same不变，若不同，则先拼接当前的count和same到字符串上，初始化count，把same移到下一个字符。

# 代码
```java
class Solution {
    public String countAndSay(int n) {
        String str="1";
        for(int i=1;i<n;i++){
            char[] c=str.toCharArray();
            String newStr="";
            int count=1; // 计数
            int index=0; // 查找的字符下标
            char same=c[index]; // 确定重复字符
            while(index<c.length){
                if(index+1==c.length){
                    newStr=newStr+count+same; // 合成字符串
                    break;
                }
                else{
                    if(c[index+1]==same){
                        count++;
                    }else{
                        newStr=newStr+count+same;
                        same=c[index+1];
                        count=1;
                    }
                    index++;
                }
            }
            str=newStr;
        }
        return str;
    }
}
```
这里可以通过在于same相同时一直添加count来加快运行时间，减少需要的内存。
```java
class Solution {
    public String countAndSay(int n) {
        String str="1";
        for(int i=1;i<n;i++){
            char[] c=str.toCharArray();
            String newStr="";
            int count=1;
            int index=0;
            char same=c[index];
            while(index<c.length){
                if(index+1==c.length){
                    newStr=newStr+count+same;
                    break;
                }
                else{
                    if(c[index+1]!=same){
                        newStr=newStr+count+same;
                        same=c[index+1];
                        count=1;
                        index++;
                    }
                    while(index+1<c.length&&c[index+1]==same){
                        count++;
                        index++;
                    }
                }
            }
            str=newStr;
        }
        return str;
    }
}
```