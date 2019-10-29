---
title: 双向绑定实现之Proxy与defineProperty
date: 2019-04-28 15:56:51
tags:
---

## 观察者模式
观察者模式在软件设计中是一个对象，维护一个依赖列表，当任何状态发生改变自动通知他们。

## 发布订阅模式
发布-订阅模式中，消息的发送方，叫做发布者，消息不会直接发送给特定的接收者，也叫订阅者。

<!--more-->

![观察者模式和发布订阅模式](./观察者模式和发布订阅模式.jpg)

## 区别
- 在观察者模式中，观察者是知道Subject的，Subject一直保持对观察者进行记录。然而，在发布订阅模式中，发布者和订阅者不知道对方的存在。他们只有通过消息代理进行通信。
- 在发布订阅模式中，组件是松耦合的，与观察者模式相反。
- 观察者模式大多数时候是同步的，比如当事件触发，Subject就会去调用观察者的方法。而发布-订阅模式大多数时候是异步的（消息队列）。
- 观察者模式需要在单个应用程序地址空间中实现，而发布-订阅模式更像交互应用模式。

## 实现双向绑定的技术方案

- KonckoutJS基于观察者模式实现双向绑定
- Ember基于数据模型实现双向绑定
- Angular基于脏检测实现双向绑定
- Vue目前是基于数据劫持实现双向绑定

## Vue双向绑定的实现原理分析

![Vue双向绑定的实现](./vue双向绑定原理.png)

- 利用Proxy或Object.defineProperty生成Observer针对对象/对象的属性进行“劫持”，在属性发生变化时通知订阅者
- 解析器Compile解析末班中的Directive（指令），收集指令所依赖的方法和数据，等待数据变化然后进行渲染
- Watcher属于Observe和Compile的桥梁，它将接受到的Observe产生的数据变化，并根据Compile提供的指令进行视图渲染，使得数据变化促使视图变化

Vue运用的数据劫持，但是依然离不开**发布订阅模式**

## 一个典型的发布订阅模式：

```javascript
var pubsub = {};

(function(ps){
	var registerLists = [],
	subId = -1;
	ps.publish = function(event, data) {
		if(!registerLists[event])
			return false;
		var subscribes = registerLists[event],
		len = subscribes ? subscribes.length : 0;
		while(len--)
			subscribes[len].callback(event,data);
		return this;
	};
	ps.subscribe = function(event, callback) {
		if(!registerLists[event])
			registerLists[event] = [];
		var token = (++subId).toString();
		registerLists[event].push({
			token: token,
			callback: callback
		});
		console.log("subscriber:"+token+" has been added!");
		return token;
	}
	ps.unsubscribe = function(token) {
		for(var key in registerLists)
			if(registerLists[key]) {
				for(var i = 0, j = registerLists[key].length; i < j; i++) {
					if(registerLists[key][i].token === token) {
						registerLists[key].splice(i, 1);
						console.log("subscriber:"+token+" has been removed!");
					}
				}
			}
	}
})(pubsub)
```
## 数据劫持

```javascript
const data = {
    key: '',
    value: ''
};
Object.keys(data).forEach( (key) => Object.defineProperty(data, key, {
    // 配置信息对象
    enumerable: true,
    configurable: true,
    get: function() {
        // do yourthings
        // 这里不能 return data[key] 会一直触发get导致堆栈溢出
        console.log('get value');
    },
    set: function(newVal) {
        // do yourthings
        // 这里不能设置对象属性的值也会一直触发set导致堆栈溢出
        console.log('set value');
    }
}))
```

所以需要用一个外部变量或者以闭包的方式去实现

```javascript
var obj = {};
var initValue = 'hello';
Object.defineProperty(obj, 'newKey', {
    get: function() {
        console.log('logger:: getter returns ', initValue);
        return initValue;
    },
    set: function(newValue) {
        console.log('logger:: old value: ', initValue, '. new value: ', newValue);
        initValue = newValue
    }
})
```

闭包的形式

```javascript
function Archiver() {
    var temperature = null;
    var archive = [];

    Object.defineProperty(this, 'temperature', {
        get: function() {
            console.log('get!');
            return temperature
        },
        set: function(value) {
            temperature = value;
            archive.push({ val: temperature });
        }
    });
    this.getArchive = function() { return archive; };
}

var arc = new Archiver();
```

### 数据劫持的优势：

1. 数据的改变直接导致视图的变化，不需要显示的调用一些方法。比如Angular的脏检查需要显示的调用markForCheck，react需要显示调用setState。
2. 可以精确的得知变化数据。而react需要额外的diff操作，来确定具体的变化，只是知道数据变化了，需要大量的diff来确认具体的变化。

### Vue数据双向绑定的实现思路（数据劫持 + 发布订阅）

1. 利用Proxy或Object.defineProperty等方法对对象/对象属性'劫持'，在属性发生变化后通知订阅者
2. Compile解析器解析模板中的Directive指令，收集指令所依赖的方法和数据，等数据变化然后在渲染
3. Watch属于Observer和Compile之间的桥梁，它将收到的Observer产生的数据变化，并根据Compile提供的指令进行视图渲染，使得数据变化促使视图变化

Object.defineProperty的第一个缺陷,无法监听数组变化。 然而Vue的文档提到了Vue是可以检测到数组变化的，但是只有以下八种方法,vm.items[indexOfItem] = newValue这种是无法检测的。

- push()
- pop()
- shift()
- unshift()
- splice()
- sort()
- reverse()

## Proxy实现双向绑定
Proxy在ES2015规范中被正式发布,它在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写

### Proxy可以直接监听对象而非属性

```javascript
const input = document.getElementById('input');
const p = document.getElementById('p');
const obj = {};

const newObj = new Proxy(obj, {
  get: function(target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function(target, key, value, receiver) {
    console.log(target, key, value, receiver);
    if (key === 'text') {
      input.value = value;
      p.innerHTML = value;
    }
    return Reflect.set(target, key, value, receiver);
  },
});

input.addEventListener('keyup', function(e) {
  newObj.text = e.target.value;
});
```

Proxy直接可以劫持整个对象,并返回一个新对象,不管是操作便利程度还是底层功能上都远强于Object.defineProperty

### Proxy可以直接监听数组的变化

```javascript
const list = document.getElementById('list');
const btn = document.getElementById('btn');

// 渲染列表
const Render = {
  // 初始化
  init: function(arr) {
    const fragment = document.createDocumentFragment();
    for (let i = 0; i < arr.length; i++) {
      const li = document.createElement('li');
      li.textContent = arr[i];
      fragment.appendChild(li);
    }
    list.appendChild(fragment);
  },
  // 我们只考虑了增加的情况,仅作为示例
  change: function(val) {
    const li = document.createElement('li');
    li.textContent = val;
    list.appendChild(li);
  },
};

// 初始数组
const arr = [1, 2, 3, 4];

// 监听数组
const newArr = new Proxy(arr, {
  get: function(target, key, receiver) {
    console.log(key);
    return Reflect.get(target, key, receiver);
  },
  set: function(target, key, value, receiver) {
    console.log(target, key, value, receiver);
    if (key !== 'length') {
      Render.change(value);
    }
    return Reflect.set(target, key, value, receiver);
  },
});

// 初始化
window.onload = function() {
    Render.init(arr);
}

// push数字
btn.addEventListener('click', function() {
  newArr.push(6);
});
```

### Proxy的其他优势

- Proxy有多达13种拦截方法,不限于apply、ownKeys、deleteProperty、has等等是Object.defineProperty不具备的。
- Proxy返回的是一个新对象,我们可以只操作新的对象达到目的,而Object.defineProperty只能遍历对象属性直接修改。
- Proxy作为新标准将受到浏览器厂商重点持续的性能优化，也就是传说中的新标准的性能红利。
- 当然,Proxy的劣势就是兼容性问题,而且无法用polyfill磨平,因此Vue的作者才声明需要等到下个大版本(3.0)才能用Proxy重写。

### 数据响应系统的实现原理

1. 在获取属性a的时候收集依赖，然后在设置属性a的时候触发之前收集的依赖。
2. 依赖收集后需要存下来
