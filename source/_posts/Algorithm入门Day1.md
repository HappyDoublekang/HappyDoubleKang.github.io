---
title: Algorithm入门Day1
date: 2019-05-05 10:27:25
tags: [Algorithm,JavaScript]
---

### Study-01

> 由简入繁的算法学习，先从简单的逻辑思维开始。

<!-- more -->

> 1.先从数学题开始，思考两个平面向量的和

向量V1 [1,2,3,4]和向量V2 [4,5,6,7] 相加会得出什么

根据数学方法计算答案应该是 V1 x V2 = 1 x 4 + 2 x 5 + 3 x 6 + 4 x 7 得到一个新的向量 如何用js的方法解答？

读完题目其实就已经得到了答案

```javascript
let v1 = [1, 2, 3, 4]
let v2 = [4, 5, 6, 7]           // 1.申明两个初始变量

function f(v1,v2) {
    var v = 0                   // 2.申明一个变量用来储存最终结果
    for (var i = 0; i < v1.length; i++){
        v += v1[i] * v2[i]      // 3.因为两个length一样所以i相同，for loop 一次就可以求和
    }
}
f(v1, v2);
```

> 2.通过上面的计算进而有了简单的编程思路以leedcode 961题 重复N次的元素为例

在大小为 2N 的数组 A 中有 N+1 个不同的元素，其中有一个元素重复了 N 次。

返回重复了 N 次的那个元素。

不妨先考虑数组中每个元素出现了几次的问题

```javascript
let numbers = ["a", "a", "c", "b"]          // 1.先假定这个元素

function count(n) {
    var obj = {}                            // 2.申明一个对象储存结果
    for (var i = 0; i < n.length; i++) {    // 3.首先考虑的一定是for loop 将每一项拿到
        var letter = n[i]                   // 4.拿到的每一项
        if (obj[letter] == undefined) {
            obj[letter] = 1                 // 5.obj开始为空obj[a]为undefined所以初始值为1
        }else{
            obj[letter] += 1                // 6.然后循环自增就得到了每一项的次数
        }
    }
    console.log(obj)
}
count(numbers)
```

回到正题，既然得到了每一项的次数那么怎么拿到最多的那个元素就有很多方法，题目中写的是2N的数组重复N次，就是长度的一半

```javascript
if (obj[letter] = n.length/2) {
    console.log(letter)
}
```

完事了





