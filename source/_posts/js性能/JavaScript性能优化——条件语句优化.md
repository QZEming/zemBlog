---
title: JavaScript性能优化——条件语句优化
abbrlink: 6202d092
date: 2019-05-25 23:49:31
tags: 性能优化
categories: JavaScript
---
与循环语句一样，条件语句也是我们在写代码中经常使用的，会用于控制代码执行的方向。最常见的条件语句就是if-else和switch，这在各种语言中都是一样的，不考虑性能方面的问题，在对这两者的使用时，如果判断条件较多的话，我们往往会使用switch，这是因为此时使用switch代码会更为易读，而在判断条件较少时，这种观点就会被反转，使用if-else看起来更为简洁。
<!-- more -->
# 对条件语句的优化
## 条件的顺序
对于条件语句的优化，首先从代码的执行顺序来看，我们将最有可能判断为true的条件放在第一个位置显然可以起到优化性能的作用，对if-else来说，假如有1000个判断条件和对应要执行的内容，但是几乎每次都是第一个条件就为true，那么后面的代码都不会对性能造成影响，反之，如果最有可能成立的条件在最后一个的话，那么几乎每次在该if-else语句执行时都会先判断1000个条件，显然会对性能造成很大的影响。

## 减少到达正确分支的条件数量
当我们要判断的条件为true的可能性都接近的时候，条件的顺序就不是非常重要了，此时要做的是减少在到达正确分支的平均判断次数，下面使用十个判断条件的例子来展示优化的方式（下面代码取自《高性能JavaScript》中的代码）
```javascript
if (value == 0) {
    return result0;
} else if (value == 1) {
    return result1;
} else if (value == 2) {
    return result2;
} else if (value == 3) {
    return result3;
} else if (value == 4) {
    return result4;
} else if (value == 5) {
    return result5;
} else if (value == 6) {
    return result6;
} else if (value == 7) {
    return result7;
} else if (value == 8) {
    return result8;
} else if (value == 9) {
    return result9;
} else {
    return result10;
}
```
在这个判断语句中，可能会判断0-10次，平均为5次，为了最小化条件判断的次数，改为下面的写法
```javascript
if (value < 6) {
    if (value < 3) {
        if (value == 0) {
            return result0;
        } else if (value == 1) {
            return result1;
        } else {
            return result2;
        }
    } else {
        if (value == 3) {
            return result3;
        } else if (value == 4) {
            return result4;
        } else {
            return result5;
        }
    }
} else {
    if (value < 8) {
        if (value == 6) {
            return result6;
        } else {
            return result7;
        }
    } else {
        if (value == 8) {
            return result8;
        } else if (value == 9) {
            return result9;
        } else {
            return result10;
        }
    }
}
```
使用这种写法后，判断的次数变为3或4次，平均也就3.5次，显然比原来少了，这种方法的关键在于给几个具体数值分区域，，再在小区域中进行具体值的判断，由此来减少判断的次数。

## 使用查找表
通过将数值或操作放在一个数组或对象，由此来省去查找的步骤，直接取值
```javascript
var results = [result0, result1, result2, result3, result4, result5, result6, result7, result8, result9, result10];
 
return results[value];
```
这里就省去了判断的过程，直接查找数组中对应的值。查找表的优点在于其时间复杂度一直为O(1)，不会因为条件增加就增加对性能的消耗，也不用写任何判断条件语句。

---
参考自《高性能JavaScript》