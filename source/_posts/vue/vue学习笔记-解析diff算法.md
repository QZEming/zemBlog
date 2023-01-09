---
title: vue学习笔记-解析diff算法
tags: vue
categories: vue
abbrlink: 702a5607
date: 2020-02-24 00:56:36
---
在vue中，DOM被抽象为一个以JavaScript对象为节点的virtual DOM(虚拟DOM)树，当数据发生改变时，会通过修改virtual DOM，使用diff算法，修改局部的内容，然后将这部分内容以打补丁的方式来修改真实的DOM，来达到视图修改的作用。
因为通过diff算法，能得出需要修改的最小单位，所以我们在更新视图的时候，只需要修改这些最小单位，这么做减小了更新的消耗。
<!-- more -->
## diff算法
---
diff算法是通过同层的树节点进行比较而非对树进行逐层搜索遍历的方式，时间复杂度只有O(n)
### diff算法流程
diff算法通过比较该层的树节点，当不同的时候，根据不同情况替换节点，当相同的时候判断新的虚拟树和原虚拟树是否都有子节点，有的话进入下一层进行比较
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190929163644283.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
图片来自：[详解vue的diff算法](https://www.cnblogs.com/wind-lanyan/p/9061684.html)

### patch
patch将新老节点进行比较，每次比较同一层的节点，patch的核心就是diff算法，通过diff算法来高效地比较新老虚拟树的差异，得出变化并修改视图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190929163322142.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
图片来自：[VirtualDOM与diff(Vue实现)](https://github.com/answershuto/learnVue/blob/master/docs/VirtualDOM%E4%B8%8Ediff(Vue%E5%AE%9E%E7%8E%B0).MarkDown)
这里相同颜色框里的节点就是比较的节点
patch的代码如下，这里要把所有部分看的一清二楚有点困难，根据上面的流程图，我们看到，要进入patchVnode的话，必须是传入的oldVnode确定为节点且与新的vnode通过sameVnode比较时返回true才可，所以下面看看sameVnode
```javascript
 /*createPatchFunction的返回值，一个patch函数*/
  return function patch (oldVnode, vnode, hydrating, removeOnly, parentElm, refElm) {
    /*vnode不存在则直接调用销毁钩子*/
    if (isUndef(vnode)) {
      if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
      return
    }

    let isInitialPatch = false
    const insertedVnodeQueue = []

    if (isUndef(oldVnode)) {
      // empty mount (likely as component), create new root element
      /*oldVnode未定义的时候，其实也就是root节点，创建一个新的节点*/
      isInitialPatch = true
      createElm(vnode, insertedVnodeQueue, parentElm, refElm)
    } else {
      /*标记旧的VNode是否有nodeType*/
      /*Github:https://github.com/answershuto*/
      const isRealElement = isDef(oldVnode.nodeType)

// *******************************看这里*********************************
      if (!isRealElement && sameVnode(oldVnode, vnode)) {
        // patch existing root node
        /*是同一个节点的时候直接修改现有的节点*/
        patchVnode(oldVnode, vnode, insertedVnodeQueue, removeOnly)
      }
// *********************************************************************


 else {
        if (isRealElement) {
          // mounting to a real element
          // check if this is server-rendered content and if we can perform
          // a successful hydration.
          if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
            /*当旧的VNode是服务端渲染的元素，hydrating记为true*/
            oldVnode.removeAttribute(SSR_ATTR)
            hydrating = true
          }
          if (isTrue(hydrating)) {
            /*需要合并到真实DOM上*/
            if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
              /*调用insert钩子*/
              invokeInsertHook(vnode, insertedVnodeQueue, true)
              return oldVnode
            } else if (process.env.NODE_ENV !== 'production') {
              warn(
                'The client-side rendered virtual DOM tree is not matching ' +
                'server-rendered content. This is likely caused by incorrect ' +
                'HTML markup, for example nesting block-level elements inside ' +
                '<p>, or missing <tbody>. Bailing hydration and performing ' +
                'full client-side render.'
              )
            }
          }
          // either not server-rendered, or hydration failed.
          // create an empty node and replace it
          /*如果不是服务端渲染或者合并到真实DOM失败，则创建一个空的VNode节点替换它*/
          oldVnode = emptyNodeAt(oldVnode)
        }
        // replacing existing element
        /*取代现有元素*/
        const oldElm = oldVnode.elm
        const parentElm = nodeOps.parentNode(oldElm)
        createElm(
          vnode,
          insertedVnodeQueue,
          // extremely rare edge case: do not insert if old element is in a
          // leaving transition. Only happens when combining transition +
          // keep-alive + HOCs. (#4590)
          oldElm._leaveCb ? null : parentElm,
          nodeOps.nextSibling(oldElm)
        )

        if (isDef(vnode.parent)) {
          // component root element replaced.
          // update parent placeholder node element, recursively
          /*组件根节点被替换，遍历更新父节点element*/
          let ancestor = vnode.parent
          while (ancestor) {
            ancestor.elm = vnode.elm
            ancestor = ancestor.parent
          }
          if (isPatchable(vnode)) {
            /*调用create回调*/
            for (let i = 0; i < cbs.create.length; ++i) {
              cbs.create[i](emptyNode, vnode.parent)
            }
          }
        }

        if (isDef(parentElm)) {
          /*移除老节点*/
          removeVnodes(parentElm, [oldVnode], 0, 0)
        } else if (isDef(oldVnode.tag)) {
          /*Github:https://github.com/answershuto*/
          /*调用destroy钩子*/
          invokeDestroyHook(oldVnode)
        }
      }
    }

    /*调用insert钩子*/
    invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
    return vnode.elm
  }
```
### sameVnode
怎么才算节点不同，diff算法中使用sameVnode来进行比较，当比较相同时才做patchVnode处理
```javascript
/*
 a b表示前后两个虚拟树的节点
 tag为标签名
 isComment表示是否为注释
 对于标签是input的节点，要求type必须相同
 */
function sameVnode (a, b) {
  return (
    a.key === b.key &&
    a.tag === b.tag &&
    a.isComment === b.isComment &&
    isDef(a.data) === isDef(b.data) &&
    sameInputType(a, b)
  )
}
// 判断input的type是否相同
function sameInputType (a, b) {
  if (a.tag !== 'input') return true
  let i
  const typeA = isDef(i = a.data) && isDef(i = i.attrs) && i.type
  const typeB = isDef(i = b.data) && isDef(i = i.attrs) && i.type
  return typeA === typeB
}
```
### patchVnode
patchVnode对于节点的处理
1. 判断新老Vnode是否都为静态且key相同，并且新的VNode是clone或者是标记了once（标记v-once属性，只渲染一次），那么只需要替换elm以及componentInstance即可；
2. 判断新老节点是否都有子节点，如果都有的且子节点不同，就对子节点使用diff操作，更新子节点
3. 如果新节点有子节点，老节点没有的话，就把老节点里面的文本内容清空，然后将新的子节点插入
4. 如果老节点有子节点，新节点没有的话，就移除这个子节点
5. 当新老节点都没有文本的时候，就直接替换文本
代码如下：
```javascript
/*patch VNode节点*/
function patchVnode (oldVnode, vnode, insertedVnodeQueue, removeOnly) {
  /*两个VNode节点相同则直接返回*/
  if (oldVnode === vnode) {
    return
  }
  /*
    如果新旧VNode都是静态的，同时它们的key相同（代表同一节点），
    并且新的VNode是clone或者是标记了once（标记v-once属性，只渲染一次），
    那么只需要替换elm以及componentInstance即可。
  */
  if (isTrue(vnode.isStatic) &&
      isTrue(oldVnode.isStatic) &&
      vnode.key === oldVnode.key &&
      (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))) {
    vnode.elm = oldVnode.elm
    vnode.componentInstance = oldVnode.componentInstance
    return
  }
  let i
  const data = vnode.data
  if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
    /*i = data.hook.prepatch，如果存在的话，见"./create-component componentVNodeHooks"。*/
    i(oldVnode, vnode)
  }
  const elm = vnode.elm = oldVnode.elm
  const oldCh = oldVnode.children
  const ch = vnode.children
  if (isDef(data) && isPatchable(vnode)) {
    /*调用update回调以及update钩子*/
    for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
    if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
  }
  /*如果这个VNode节点没有text文本时*/
  if (isUndef(vnode.text)) {
    if (isDef(oldCh) && isDef(ch)) {
      /*新老节点均有children子节点，则对子节点进行diff操作，调用updateChildren*/
      if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
    } else if (isDef(ch)) {
      /*如果老节点没有子节点而新节点存在子节点，先清空elm的文本内容，然后为当前节点加入子节点*/
      if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
      addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
    } else if (isDef(oldCh)) {
      /*当新节点没有子节点而老节点有子节点的时候，则移除所有ele的子节点*/
      removeVnodes(elm, oldCh, 0, oldCh.length - 1)
    } else if (isDef(oldVnode.text)) {
      /*当新老节点都无子节点的时候，只是文本的替换，因为这个逻辑中新节点text不存在，所以直接去除ele的文本*/
      nodeOps.setTextContent(elm, '')
    }
  } else if (oldVnode.text !== vnode.text) {
    /*当新老节点text不一样时，直接替换这段文本*/
    nodeOps.setTextContent(elm, vnode.text)
  }
  /*调用postpatch钩子*/
  if (isDef(data)) {
    if (isDef(i = data.hook) && isDef(i = i.postpatch)) i(oldVnode, vnode)
  }
}
```
### updateChildren
updateChildren是diff算法中具体对节点的操作，通过sameVnode来判断是否要进行操作，对不同节点的判断结果决定使用什么操作
（下面的图片均来自[VirtualDOM与diff(Vue实现)](https://github.com/answershuto/learnVue/blob/master/docs/VirtualDOM%E4%B8%8Ediff(Vue%E5%AE%9E%E7%8E%B0).MarkDown)）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190929183255635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
updateChildren方法在一开始首先在old VNode和new VNode首尾共设置了四个指针，通过比较这四个指针的值和new VNode与old VNode中的指针来进行不同的操作，总的有六种情况
(为了方便，下面将oldStartIdx称为oS，oldEndIdx称为oE，下面两个指针称为nS和nE)
#### 比较情况
1. 比较oS和nS
比较的时候都是使用sameVnode来进行比较，如果返回true的话，就对这两个指针指向的节点使用patchVnode操作，然后两个指针都向后，如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190929183929611.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)

2. 比较oE和nE
oE和nE的比较与第一种情况类似，对sameVnode返回true的两个节点使用patchVnode操作，然后两个指针都向前移动

3. 比较oS和nE
oS和nE比较，如果sameVnode返回true，那么将当前oS指向的节点，移动到当前oE指向的节点所在的位置的后面，同时对该节点和nE指向的节点使用patchVnode操作，oS的指针向后移动，nE的指针向前移动
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190929184314979.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
4. 比较oE和nS
当oE和nS使用sameVnode返回true时，将oE指向的节点放到oS指向的节点的前面，对该节点和nS指向的节点使用patchVnode操作，然后oE指针向前移动，nS指针向后移动

5. nS和oS到oE之间的节点比较（为true）
当上面的四种情况通过sameVnode都没法返回true的时候，则会创建一个key为旧的vNode，value为vNode对应index的哈希表，然后使用nS指向的vNode和哈希表中的vNode比较，为true的时候，将这个vNode（图中的elmToMove指针指向的vNode）放到oS指向的vNode的前面，同时对该vNode和nS指向的vNode进行patchVnode操作，然后nS指针向右移动
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190929185117625.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
6. nS和oS到oE之间的节点比较（为false）
同上面第5的情况，但是我们在oS和oN之间也找不到和nS指向的vNode相同的vNode，所以此时我们要调用createElm来新建一个vNode，并将其插入到oS指向的vNode之前的位置，并对其和nS指向的vNode使用patchVnode操作，随后nS指针向右移动
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190929185339847.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
#### 结束情况
结束有两种情况
1. oS>oE
当oS>oE，即是说老的节点比新的节点少，所以接下来就要将剩余的节点插入，插入的位置在oS后面
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190929190028295.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
2. nS>nE
如果nS>nE，那么说明新节点已经遍历完了，而老节点还没遍历完，那么还没遍历到的老节点，也就是新节点中没有的，对于这部分节点，我们可以直接丢弃掉
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019092919012942.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
下面是代码的实现
```javascript
function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    let oldStartIdx = 0
    let newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, elmToMove, refElm

    // removeOnly is a special flag used only by <transition-group>
    // to ensure removed elements stay in correct relative positions
    // during leaving transitions
    const canMove = !removeOnly

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx]
      } else if (sameVnode(oldStartVnode, newStartVnode)) {
        /*前四种情况其实是指定key的时候，判定为同一个VNode，则直接patchVnode即可，分别比较oldCh以及newCh的两头节点2*2=4种情况*/
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)
        oldStartVnode = oldCh[++oldStartIdx]
        newStartVnode = newCh[++newStartIdx]
      } else if (sameVnode(oldEndVnode, newEndVnode)) {
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue)
        oldEndVnode = oldCh[--oldEndIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue)
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
        oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue)
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
        oldEndVnode = oldCh[--oldEndIdx]
        newStartVnode = newCh[++newStartIdx]
      } else {
        /*
          生成一个key与旧VNode的key对应的哈希表（只有第一次进来undefined的时候会生成，也为后面检测重复的key值做铺垫）
          比如childre是这样的 [{xx: xx, key: 'key0'}, {xx: xx, key: 'key1'}, {xx: xx, key: 'key2'}]  beginIdx = 0   endIdx = 2  
          结果生成{key0: 0, key1: 1, key2: 2}
        */
        if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        /*如果newStartVnode新的VNode节点存在key并且这个key在oldVnode中能找到则返回这个节点的idxInOld（即第几个节点，下标）*/
        idxInOld = isDef(newStartVnode.key) ? oldKeyToIdx[newStartVnode.key] : null
        if (isUndef(idxInOld)) { // New element
          /*newStartVnode没有key或者是该key没有在老节点中找到则创建一个新的节点*/
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)
          newStartVnode = newCh[++newStartIdx]
        } else {
          /*获取同key的老节点*/
          elmToMove = oldCh[idxInOld]
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !elmToMove) {
            /*如果elmToMove不存在说明之前已经有新节点放入过这个key的DOM中，提示可能存在重复的key，确保v-for的时候item有唯一的key值*/
            warn(
              'It seems there are duplicate keys that is causing an update error. ' +
              'Make sure each v-for item has a unique key.'
            )
          }
          if (sameVnode(elmToMove, newStartVnode)) {
            /*Github:https://github.com/answershuto*/
            /*如果新VNode与得到的有相同key的节点是同一个VNode则进行patchVnode*/
            patchVnode(elmToMove, newStartVnode, insertedVnodeQueue)
            /*因为已经patchVnode进去了，所以将这个老节点赋值undefined，之后如果还有新节点与该节点key相同可以检测出来提示已有重复的key*/
            oldCh[idxInOld] = undefined
            /*当有标识位canMove实可以直接插入oldStartVnode对应的真实DOM节点前面*/
            canMove && nodeOps.insertBefore(parentElm, newStartVnode.elm, oldStartVnode.elm)
            newStartVnode = newCh[++newStartIdx]
          } else {
            // same key but different element. treat as new element
            /*当新的VNode与找到的同样key的VNode不是sameVNode的时候（比如说tag不一样或者是有不一样type的input标签），创建一个新的节点*/
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)
            newStartVnode = newCh[++newStartIdx]
          }
        }
      }
    }
    if (oldStartIdx > oldEndIdx) {
      /*全部比较完成以后，发现oldStartIdx > oldEndIdx的话，说明老节点已经遍历完了，新节点比老节点多，所以这时候多出来的新节点需要一个一个创建出来加入到真实DOM中*/
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
      /*如果全部比较完成以后发现newStartIdx > newEndIdx，则说明新节点已经遍历完了，老节点多余新节点，这个时候需要将多余的老节点从真实DOM中移除*/
      removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
    }
  }
```

参考文章：
[VirtualDOM与diff(Vue实现)](https://github.com/answershuto/learnVue/blob/master/docs/VirtualDOM%E4%B8%8Ediff(Vue%E5%AE%9E%E7%8E%B0).MarkDown)
[详解vue的diff算法](https://www.cnblogs.com/wind-lanyan/p/9061684.html)