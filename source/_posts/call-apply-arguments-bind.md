---
title: 'call,apply,arguments,bind'
date: 2019-05-20 14:20:31
tags: [JavaScript]
---
### call & apply
该方法的语法和作用与 apply() 方法类似，只有一个区别，就是 call() 方法接受的是一个参数列表，而 apply() 方法接受的是一个包含多个参数的数组。

```javascript
fun.call(thisArg, arg1, arg2, ...)
```

<!--more-->

**thisArg**
在 fun 函数运行时指定的 this 值。需要注意的是，指定的 this 值并不一定是该函数执行时真正的 this 值，如果这个函数在非严格模式下运行，则指定为 null 和 undefined 的 this 值会自动指向全局对象（浏览器中就是 window 对象），同时值为原始值（数字，字符串，布尔值）的 this 会指向该原始值的自动包装对象。

```javascript
function getLength () { return this.length }
getLength.call('abc') // 3
```
#### call & apply 的主要应用场景

##### 调用函数，传递参数

```javascript
function addTwo (x, y) {
    return x + y;
}

function addThreeCall (x, y, z) {
    return addTwo.call(this, x, y) + z;
}

function addThreeApply (x, y, z) {
    return addTwo.apply(this, [x, y]) + z;
}
```

##### 改变函数作用域

```javascript
var name = 'zy';

var Obj = { name: 'zhongyan' };

function sayName (name) = {
    return this.name;
}

console.log(sayName.call(this));  // zy
console.log(sayName.call(Obj));  // zhongyan
```

##### 实现继承

```javascript
function Person () {
    this.sayName = function () {
        return this.name;
    }
}
function Chinese (name) {
    Person.call(this);
    this.name = name;

    this.ch = function () {
        alert('中国人');
    }
}

var chinese = new Chinese('小龙');
console.log(chinese.sayName());
```

#### Polyfill
##### call Polyfill
```javascript
Function.prototype._call = function (context) {
    const me = this;
    if (typeof me !== 'function')
    throw Error('only apply to function')

    const ctx = context || window
    ctx.method = me
    const arg = [...arguments].slice(1)
    const result = ctx.method(...arg)
    delete ctx.method
    return result
}
```
**例子**
```javascript
function sayName () { console.log(this.name) }
var obj = { name: 'zhongyan' }
sayName._call(obj)
```

- 首先将被调用函数的的this (该方法本身)存下来。
- 检查this是否是函数，因为call方法只用在函数上，如果不是函数则抛出Error
- 将被调用的函数本身（this）赋值到调用者context的临时方法method上
- 取出参数，执行context的临时方法method
- 返回结果
- 删除调用者的临时方法method

##### applyPolyfill

```javascript
if (!Function.prototype.call || !Function.prototype.apply || !Function.prototype.bind) {
    // TODO: 添加Polyfill
}
```

```javascript
Function.prototype._apply = function (context, args) {
    const me = this
    if (typeof me !== 'function')
    throw Error('only apply to function')

    const ctx = context || window
    ctx.method = me
    const result = ctx.method(...args)
    delete ctx.method
    return result
}
```
和call的实现基本一致，只是参数是数组形式了。

### bind

#### Polyfill

##### bindPolyfill

```javascript
Function.prototype._bind = function () {
    const me = this
    if (typeof me !=== 'function')
        throw Error('only apply to function')
    const [ctx, ...args] = [...arguments]
    const result = function () {
        const partialArgs = [...arguments]
        if (this instanceof result)  // new 的情况，所以这种情况我们需要忽略传入的this
            return me(...args, ...partialArgs)
        return me.apply(ctx, args.concat(partialArgs))
    }
    return result
}
```




