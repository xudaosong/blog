JavaScript的这些概念都掌握了么
============

## 目录 {ignore=true}
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [1. 变量提升](#1-变量提升)
* [2. 函数提升](#2-函数提升)
* [3. 暂时性死区](#3-暂时性死区)
* [4. 高阶函数](#4-高阶函数)
* [5. 柯里化](#5-柯里化)
* [6. 函数节流](#6-函数节流)
* [7. 分时函数](#7-分时函数)
* [8. 纯函数](#8-纯函数)
* [6. 其它的一些知识点](#6-其它的一些知识点)

<!-- /code_chunk_output -->

## 1. 变量提升
变量提升是指在还没定义变量前就使用了该变量，因为编译器把变量声明提升到它所在作用域的最开始的部分。
```javascript
// 使用var的情况
console.log(foo); // 输出: undefined
var foo = 2;

// 使用let的情况
console.log(foo); // 输出: ReferenceError: foo is not defined
let foo = 2;
```
上面代码中
1. 变量foo用var命令声明，会发生变量提升，因为在脚本开始运行时，变量声明会提升到该变量作用域的头部，但还未赋值，因此输出undefined。
2. 变量foo用let命令声明，不会发生变量提升。
因此在使用es6编码时，这是推荐使用let或const的定义变量的理由之一。

举个栗子：
```javascript
var foo = 2;
function print(){
    console.log(foo);
    var foo = 0;
}
print(); // 输出: undefined
```
上面代码中，其实我们想要输出的是2，但实际上我们输出的是undefined。因为print()函数内部也定义了foo变量，编译器在执行该方法时把变量声明提升到print()函数体的头部，因此输出的是undefined。
编译器执行的顺序如下：
```javascript
var foo = 2;
function print(){
    var foo; // 进行了变量提升
    console.log(foo);
    foo = 0;
}
print(); // 输出: undefined
```
## 2. 函数提升
与变量提升一样，函数也存在类似的提升问题，但函数提升只限于函数声明的形式。
函数有两种创建方式
* 函数声明 （会进行函数提升）
* 函数表达式（不会进行函数提升）
```javascript
foo('Will');
function foo(name){
    console.log(name); //输出: Will
}
```
上面代码，由于采用函数声明语法，因此正常输出Will。因为编译器执行的顺序如下：
```javascript
function foo(name){
    console.log(name); //输出: Will
}
foo('Will');
```
从上面代码可以看出，函数的声明被提升到该作用域的顶部。
```javascript
foo('Will');
var foo = function(name){
    console.log(name); // 输出: TypeError: foo is not a function
}
```
上面代码，采用了函数表达式语法，不会进行函数声明提升，因此提示foo函数未定义。
再举个粟子：
```javascript
var getName = function() {
    console.log(1);
}
function getName () {
    console.log(2);
}
getName();  // 输出: 1
```
上面代码涉及变量提升和函数提升。首先使用函数声明`function getName(){}`会被提升到顶部。然后也会把getName的变量声明提升到顶部，但变量的赋值保留在原来的位置。因此编译器执行的顺序如下：
```javascript
var getName;
function getName () {
    console.log(2);
}
getName = function() {
    console.log(1);
};
getName();  // 输出: 1
```
## 3. 暂时性死区
暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。这是为了防止在变量声明前就使用该变量，从而导致意料之外的行为。
```javascript
let foo = 2;
function print(){
    console.log(foo);
    let foo = 0;
}
print(); // 输出ReferenceError: foo is not defined
```
上面代码中，print()函数不会继承父级作用域的foo变量。因为print()函数体内定义了foo变量，这时该块级作用域的foo变量就“绑定”这个区域，不再受外部的影响。所以在print()函数体内声明foo变量之前，使用了foo变量，就会报错。
总之，在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称TDZ）。
另外，在es6中，除了`let`之外`const`也存在暂时性死区。

## 4. 高阶函数
高阶函数是把函数作为参数，或是将函数作为返回值的函数，如下面代码所示：
```javascript
function foo(x){
    return function() {
        return x;
    };
}
```
高阶函数比普通函数提供了更好的灵活性。举个数组的sort()方法示例：
```javascript
var points = [30,50,5,32,2]
points.sort(function(a,b){
    return a-b;
})
// 输出： [2, 5, 30, 32, 50]
```
上面的示例可以通过变更sort()方法的参数，决定不同的排序方式，从而提供了更好的复用和更高灵活性。
回调函数是高阶函数的子集，因为回调函数，一般是将函数做为参数。而高阶函数还可以返回一个函数。
## 5. 柯里化
柯里化(Currying)，又译为卡瑞化或加里化，是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。简单的说：只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。
```javascript
fn(1,2,3,4) => fn(1)(2)(3)(4)
```
为什么我们不使用多参数函数，而要进行柯里化？
函数柯里化允许和鼓励你分隔复杂功能变成更小更容易分析的部分。这些小的逻辑单元显然是更容易理解和测试的，然后你的应用就会变成干净而整洁的组合，由一些小单元组成的组合。

柯里化的作用：
* 提高通用性
* 延迟执行
* 参数复用

dva/redux的connect就是一种应用

## 6. 函数节流
## 7. 分时函数
## 8. 纯函数
## 6. 其它的一些知识点
* const实际上保证的并不是变量的值不得改动，而是变量指向的那个内存地址不得改动。所以对象、数组只要引用不变，则是可以改动的。如果要不改动，可以使用Object.freeze
* 目前阶段，通过 Babel 转码，CommonJS 模块的require命令和 ES6 模块的import命令，可以写在同一个模块里面，但是最好不要这样做。因为import在静态解析阶段执行，所以它是一个模块之中最早执行的。
* import命令具有提升效果，会提升到整个模块的头部，首先执行