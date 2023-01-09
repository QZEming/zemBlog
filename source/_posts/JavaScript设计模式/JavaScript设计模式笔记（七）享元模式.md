---
title: JavaScript设计模式笔记（七）享元模式
abbrlink: 8c2c15c6
date: 2019-05-11 23:51:37 
tags: JavaScript设计模式
categories: JavaScript
---
# 享元模式的定义
从名字来看，享--->共享，元--->对象，这里的享元指的是共享对象的意思。在享元模式中，我们尽可能减少对象的使用。
<!-- more -->
# 享元模式的通用结构
使用享元模式的关键在于确定对象间的共性和在特定场景的不同，我们所需要的对象数量就是特定场景的数量，在同一特定场景下我们只需要一个相同的对象。我们将共性称为内部状态，而在特定场景不同的特性称为外部状态。

以下面的例子来理解享元模式。现在工厂进了一批衣服，需要有模特来穿上衣服来为每款衣服拍摄广告，（假设衣服的尺码都是相同的），现在有10款男装和10款女装，我们可以每款衣服都找一个模特来穿，但是显然这样太费钱了，事实上我们只需要一个男模特和一个女模特。在代码层面上来说，每款衣服对应创建一个对象太耗费内存了，我们只需要创建两个对象就可以解决这个问题，而这两个对象的不同在于它们的sex属性不同。
```javascript
let model = function(sex) {
    return {
        sex: sex,
        wear: function(clothes) {
            console.log(`the ${sex} model wears ${clothes}`);
        }
    }
}
 
let maleModel = model('male');
let femaleModel = model('female');
 
for (let i = 1; i <= 10; i++) {
    maleModel.wear(`maleClothes${i}`)
}
// the male model wears maleClothes1
// the male model wears maleClothes2
// the male model wears maleClothes3
// the male model wears maleClothes4
// the male model wears maleClothes5
// the male model wears maleClothes6
// the male model wears maleClothes7
// the male model wears maleClothes8
// the male model wears maleClothes9
// the male model wears maleClothes10
 
for (let i = 1; i <= 10; i++) {
    femaleModel.wear(`femaleClothes${i}`)
}
// the female model wears femaleClothes1
// the female model wears femaleClothes2
// the female model wears femaleClothes3
// the female model wears femaleClothes4
// the female model wears femaleClothes5
// the female model wears femaleClothes6
// the female model wears femaleClothes7
// the female model wears femaleClothes8
// the female model wears femaleClothes9
// the female model wears femaleClothes10
```
在上面的代码中，存在两个问题：1.我们可能会不小心创建两个相同的对象；2.我们在还没用到对象的时候就创建了对象；

要解决上面两个问题，1.可以使用单例模式来解决；2.在调用方法的时候再创建对象
```javascript
let model = (function() {
    let models = []
    return {
        create: function(sex) {
            let i = 0;
            for (; i < models.length; i++) { // 遍历现有的model对象
                if (models[i].sex === sex) // 判断是否已经有这种类型的model对象
                    return models[i]
            }
            models.push({ // 没有的时候推入一个model对象
                sex: sex,
                wear: function(clothes) {
                    console.log(`the ${sex} model wears ${clothes}`);
                }
            })
            return models[i]
        },
        wear: function(sex, clothes) {
            return this.create(sex).wear(clothes);
        }
    }
})()
 
for (let i = 1; i <= 10; i++) {
    model.wear('male', `clothes${i}`)
}
 
for (let i = 1; i <= 10; i++) {
    model.wear('female', `clothes${i}`)
}
// 执行结果和上面代码一样
```
在上面的代码中，每次遍历已有的模型，如果没有需要的模型的话，就推入一个新的模型，通过这种做法，上面的代码实际上只使用了两个对象。

# 对象池
对象池与享元模式类似，都可以做到性能优化，但是在实现上有点不一样。

对象池存放着空闲对象，当我们需要对象时，会先从对象池中寻找对象，若对象池中没有对象，再新建对象。上面第二段代码中的models类似于对象池，但有一点不同的是没有体现出占用，因为上面的例子中对象是被依次使用的，当上一个对对象的使用结束后才继续寻找对象使用，我们换一个例子来看。假如当前要同时拍四件衣服的广告，当这四件衣服广告拍完后，我们又需要同时拍六件衣服的广告，为了减少麻烦，我们会让第一次拍四件广告的四个模特留下来，我们再去找另外两个模特。换成代码层面来说，就是我们将四个已经新建的对象放到对象池里，然后再新建两个对象完成结果。
```javascript
let models = (function() {
    let models = [];
    let count = 0; // 计算已创建的对象数量
    return {
        create: function() {
            if (models.length === 0) {
                count++;
                console.log(`the object has created ${count} objects`);
                return {
                    wear: function(clothes) {
                        console.log(`the model wears ${clothes}`);
                    },
                    recover: function() {
                        models.push(this);
                    }
                }
            }
            return models.shift()
        },
        wear: function(clothes) {
            return this.create().wear(clothes);
        }
    }
})()
 
for (let i = 1; i <= 4; i++) {
    let model = models.create();
    model.wear(`clothes${i}`);
    setTimeout(() => {
        model.recover()
    }, 500)
}
// the object has created 1 objects
// the model wears clothes1
// the object has created 2 objects
// the model wears clothes2
// the object has created 3 objects
// the model wears clothes3
// the object has created 4 objects
// the model wears clothes4
 
setTimeout(() => {
    for (let i = 5; i <= 10; i++) {
        let model = models.create();
        model.wear(`clothes${i}`);
    }
}, 1000)
 
// the model wears clothes5
// the model wears clothes6
// the model wears clothes7
// the model wears clothes8
// the object has created 5 objects
// the model wears clothes9
// the object has created 6 objects
// the model wears clothes10
```
这里假设在500ms完成第一组拍摄（现实当然不可能这么快，我只是懒得等结果。。。），这里一共创建了四个对象，在第1000ms时进行第二组拍摄，在这组拍摄中，只创建了两个对象，就是因为引用了对象池里的对象。

# 享元模式的优点
很明显享元模式能减少对象的构建，对于解决系统内存不足来说非常有效

# 享元模式的缺点
额。暂时没想到，可能就是在构建的时候需要先去区分内部状态和外部状态，在本来所需对象就少的情况下没必要使用。

参考自曾探的《JavaScript设计模式与开发实践》