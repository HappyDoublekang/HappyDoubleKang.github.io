---
title: ES6模板字符串
date: 2019-03-13 09:45:40
tags: 模板字符串
---

## 为了能够让我们更加方便的拼接，操作字符串，ES6出了模板字符串。

<!-- more -->

模板字符串是用`反引号代替常规字符串的'单引号或者"双引号，从而让字符串拥有可以将一部分替换成变量拼接在一起能力，当然，模板字符串不仅仅如此。

## 基本语法

### 先看一个简单的例子

```javascript
let message = `hello`;
console.log(message); // "hello" 
console.log(typeof message); // "string"
console.log(message.length); // 5
```

通过上述例子，可以看出，其实模板字符串创建的就是普通的字符串，没有什么特别的地方。当然，如果想**创建多行的字符串**，以前的方法可能是这样：

```javascript
var text = "hello\nworld";
console.log(text);  //hello
                    //world
```

而如果使用模板字符串，就可以直接换行写，或者用\n：

```javascript
var text =`hello\nworld`;
var text1 =`hello 
world`;
console.log(text);  //hello 
                    //world 
console.log(text1); //hello 
                    //world
```

使用模板字符串还有值得注意的几点：

**1.空格符（空白符）也会占位置：**

```javascript
var text=`hello
          world`;
console.log(text);           //hello
                             //world
console.log(text.length)     //27
```

**2.如果想让\n输出，可以使用转义字符，变为\\\n，或者直接使用String.raw:**

```javascript
var text =String.raw`hello\nworld`;
var text1 =`hello\\nworld`;
console.log(text);              //hello\nworld 
console.log(text1);             //hello\nworld 
```

## 进入正题

之前说介绍的功能只能说是普通字符串的改良版，下面介绍真正的模板字符串强大的地方，可以**嵌入变量**而且写法超级简单，具体代码如下：

```javascript
let myName = "moonburn";
let message = `hello,my name is ${myName}`;
console.log(message)           //"hello,my name is moonburn"
```
=>对比老版写法
```javascript
var myName = "moonburn";
var message = 'hello,my name is '+ myName;
console.log(message)           //"hello,my name is moonburn"
```

当然，模板字符串接受的不仅仅是变量，还可以是函数回调，简单的计算，比如：

```javascript
let jiao= 0.1,
message = `${jiao}角等于${jiao*10}分，也等于${(jiao* 0.1).toFixed(3)}元。`
console.log(message)           //"0.1角等于1分，也等于0.010元。"
```

甚至是模板字符串本身：

```javascript
let myName = "moonburn";
let message = `hello,${`my name is ${myName}`}`;
console.log(message)           //"hello,my name is moonburn"
```

不仅如此，ES6还贴心的出了模板标签（Tagged Templates）。

## 模板标签

声明一个模板标签：

```javascript
let text = sayHello`hello`;
function sayHello(){
    //do something
}
```

正如你所看到的，sayHello就是模板标签，那么这个到底是干什么的呢？
可以看到，下面，我们声明的一个和模板标签同名的函数，这个函数就是模板标签的本体！具体有什么用呢？看下面这个例子：

```javascript
let text = sayHello`hello`; 
function sayHello(array1,...array2){ 
    console.log(array1); 	//["hello"] 
    console.log(array2); 	//undefined return 123;
 } 
console.log(text) 	        //123
```

是不是有点明白了？嗯，那我们继续深一步探讨，其实这个模板标签也就是sayHello把模板字符串hello当做参数（两个参数具体怎么分配，我们下面在探讨），然后经过一系列的计算，最后返回一个字符串return 123，给text,所以最后text的值是123。
下面我们再看一个复杂一点的例子：

```javascript
let word= "world" 
let name = "moonburn" 
let text = sayHello`hello ${word},my name is ${name}`; 
function sayHello(array1,...array2){ 
    console.log(array1);	 //["hello ",",my name is ",""] 
    console.log(array2); 	 //["world", "moonburn"] 
    return 123;
} 
console.log(text) 	         //123
```

通过这个例子，我们可以明白，函数的第一个参数，就是模板字符串中非变量的部分组成的数组，值得注意的是，模板字符串的最开始和最后一定要是普通字符串，如果是变量开始或者结尾，就像上述例子，那么就会有空字符串也就是""出现，通过这一点，我们就可以保证，第一个参数的长度永远比第二个参数长一个单位，也就是说array1.length === array2.length+1恒为true。第二个参数是什么呢？显而易见了，就是变量。那么我们来看看默认的模板标签是怎么样的呢？

```javascript
let word= "world" 
let name = "moonburn" 
let text = sayHello`hello ${word},my name is ${name}`; 
function sayHello(array1,...array2){ 
    let result = ""; 
    for (let i = 0; i < array2.length; i++) { 
        result += array1[i]; 
        result += array2[i];
    } 	//加上最后一个普通字符串。 
    result += array1[array1.length - 1]; 
    return result; 
} 
console.log(text) 
        //"hello world,my name is moonburn"
```

当然，array2的值不仅仅只是字符串，如果是需要计算的数，或者回调函数，那么计算完毕之后再进行模板标签的函数调用。

```javascript
let word= "world" 
let name = function(){ 
    console.log("函数调用") 
    return "moonburn" 
} 
let text = sayHello`hello ${word},my name is ${name()}`; 
function sayHello(array1,...array2){ 
    let result = ""; 
    for (let i = 0; i < array2.length; i++) { 
        result += array1[i]; 
        result += array2[i]; 
    } 	            //加上最后一个普通字符串。 
    result += array1[array1.length - 1]; 
    return result; 
} 
console.log(text) 	//函数调用 //"hello world,my name is moonburn"
```

## 总结

不带变量的模板字符串就是普通字符串的加强版，换行之类的操作更加方便，注意空格。
带变量的模板字符串可以接受变量计算，函数回调等，最后输出的还是字符串。
模板标签可以自己人性化配置模板字符串，接受2个参数，返回值也是自己定义。

作者：moonburn
链接：https://www.jianshu.com/p/1bf65c57b42e
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。


