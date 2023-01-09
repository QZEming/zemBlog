---
title: LeetCode45.跳跃游戏IIJavaScript实现
abbrlink: d3ad7318
date: 2020-01-22 19:22:49
tags: leetcode
categories: 算法
---
给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

你的目标是使用最少的跳跃次数到达数组的最后一个位置。
<!-- more -->
示例:

输入: [2,3,1,1,4]
输出: 2
解释: 跳到最后一个位置的最小跳跃数是 2。
从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/jump-game-ii
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

对于这道题，我使用了两种解法

# 1. 直接寻找到达目标位置的最近点
对于当前位置，当前位置对应的值就是我们能到达的最远值，而我们只要从左到右找能一下子跳到目标值的位置，那个位置就是我们的下一个目标值，以示例输入为例
2，3，1，1，4
在一开始，我们设step=0，我们的目标值就是下标为4的最后一个位置，即target=4，从左到右遍历，在第一个位置index=0无法跳到目标位置，所以我们接下去看，第二个位置index=1可以一下子跳到目标位置，所以我们将step++，目标值target=1，
重新遍历，这次可以直接从第一个位置index=0跳到目标位置，所以我们将step++，然后target=0，因为已经是0了，所以输出，答案为2
代码如下
```javascript
var jump = function(nums) {
    let target = nums.length-1;
    let step = 0;
    while(target !=0){
        for(let i=0;i<target ;i++){
            if(nums[i]>=target -i){ // 可以跳到目标位置了
                target =i;
                step++;
                break;
            }
        }
    }
    return step;
};
```
时间复杂度：O(n*n)

# 2.贪心算法
上面的方法虽然可行，但是O(n*n)的时间复杂度太高了，所以我改成使用贪心算法来解决
在每个当前位置，判断当前位置能跳到的位置中，哪个位置可以跳的更远，就跳到那个位置，直到找到某个位置可以跳到目标值就停下，以示例输入为例
2，3，1，1，4
一开始step=0，在index=0的位置，当前可以跳的是index1=1和index2=2的位置，而index1的位置可以跳的最远距离是index1+nums[index1]=4，而index2可以跳的最远距离是index2+nums[index2]=3，所以我们跳到index1的位置，step++
到了index1的位置，我们判断当前可以跳到的位置已经超过了目标位置target=4，所以我们跳出遍历，step++，返回step即是2
代码如下
```javascript
var jump = function(nums) {
    let target = nums.length-1;
    let step = 0;
    let index = 0;
    while(index<target&&index+nums[index]<target){
        let max = index+1; // 下一步到的位置可以到达的最远位置
        let maxI = index+1; // 当前找到的可以在下一步跳的更远的位置
        step++;
        for(let i =1;i<=nums[index];i++){
            if(index+i+nums[index+i]>max){ // 找到可以跳的更远的位置
                max=index+i+nums[index+i];
                maxI=index+i;
            }
        }
        index = maxI;
    }
    if(index<target)
        step++;
    return step;
};
```
使用贪心算法时间复杂度为O(n)