---
title: JavaScript文档碎片的使用
abbrlink: ee22ea8c
date: 2020-01-27 12:57:21
tags: JavaScript
categories: JavaScript
---
JavaScript中，文档碎片独立于DOM树之外，存在于内存中，在创建之初为一个空白的文档片段，我们可以使用createDocumentFragment来创建
<!-- more -->
```javascript
let fragment = document.createDocumentFragment();
```
对文档碎片的操作，包括插入节点，删除节点的API都和其他DOM元素相同，但是也存在一些不同
1. 将DOM树中的元素插入到文档碎片中的时候，DOM树中的元素会删除
```html
<body>
    <div id="t1">t1</div>
    <button onclick="insert()">insert</button>
    <script>
        function insert() {
            let frag = document.createDocumentFragment()
            frag.appendChild(document.querySelector("#t1"))
            console.log(frag)
        }
    </script>
</body>
```
这里，点击按钮，会发现页面中节点消失了，而出现在打印的frag中
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200127123806323.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/20200127123844755.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
2. 当我们将文档碎片插入DOM树中时，插入的不是文档碎片，而是文档碎片的子孙元素
```html
<body>
    <div id="t1">t1</div>
    <button onclick="insert()">insert</button>
    <script>
        function insert() {
            let frag = document.createDocumentFragment()
            let p = document.createElement('p')
            p.innerHTML = 'p'
            let div = document.createElement('div')
            div.innerHTML = 'div'
            frag.appendChild(p)
            frag.appendChild(div)
            document.querySelector('#t1').appendChild(frag)
        }
    </script>
</body>
```
看看点击按钮后的节点，插入的是文档碎片的子孙元素
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200127124332779.png)
3. 创建节点时无需传入要创建的是什么节点
上面也说了，一开始创建的文档碎片就是一个空白的文档片段，而我们使用```document.createElement()```创建元素时，是需要传入字符串来表示要创建的标签的tagName的，而文档碎片不必传入标签名，实际上也不需要，因为最后插入的也只是文档碎片的子孙节点

---
那么，文档碎片有什么实用性呢，上面也说到了，文档碎片不放在DOM树中，而是放在内存中，这就意味着我们将节点替换插入文档碎片中的时候不会引起回流重绘，如果我们要插入大量节点，一个个插入必然会引起大量的回流和重绘，先放到文档碎片中，再将文档碎片插入，可以减少回流重绘，提高性能，但实际上是不是能提高呢，我测试了一下
```html
<body>
    <button onclick="direct()">直接插入</button>
    <button onclick="el()">放到元素中再插入</button>
    <button onclick="frag()">放到文档碎片后再插入</button>
    <ul id="test1"></ul>
    <ul id="test3"></ul>
    <script>
        function direct() {
            console.time('直接插入')
            for (let i = 0; i < 100000; i++) {
                let li = document.createElement('li');
                li.innerHTML = i;
                document.querySelector('#test1').appendChild(li)
            }
            console.timeEnd('直接插入')
        }

        function el() {
            console.time('放到元素中再插入')
            let ul = document.createElement('ul');
            for (let i = 0; i < 100000; i++) {
                let li = document.createElement('li');
                li.innerHTML = i;
                ul.appendChild(li)
            }
            document.body.appendChild(ul)
            console.timeEnd('放到元素中再插入')
        }

        function frag() {
            console.time('放到文档碎片后再插入')
            let frag = document.createDocumentFragment()
            for (let i = 0; i < 100000; i++) {
                let li = document.createElement('li');
                li.innerHTML = i;
                frag.appendChild(li)
            }
            document.querySelector('#test3').appendChild(frag)
            console.timeEnd('放到文档碎片后再插入')
        }
    </script>
</body>
```
测试之后发现打印出来的时间差别不大，甚至有时出现了放入文档碎片反而更久的情况
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200127124817834.png)
所以性能优化的也只是理论上的，影响性能的因素还有很多