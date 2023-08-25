---
title: 探索前端的 this 指向
date: 2023.09.01
tags:
- JS
- this
  categories: Technology   
  mathjax: true
  comments: true
  toc: true
  description: 本文将深入探讨 this 在前端开发中的应用场景以及不同情况下的指向规则，更好地理解和运用 this 指向。
---

在前端开发中，this 是一个常见的概念。它代表了当前执行上下文中的对象或函数，并且在不同的情况下，this 的指向也会有所不同。

本文将深入探讨 this 在前端开发中的应用场景以及不同情况下的指向规则，更好地理解和运用 this 指向。

## 拿个栗子
```js
uni.showModal({
  //...
  success: function(res) {
    if (res.confirm) {
        this.$request('/mini/shop/update', 'POST', {
        //...
      }).then(res => {
        //...
      });
    }
  }
});
```

执行上面代码，出现了 `this.$request is not a function` 的错误。

```
TypeError: this.$request is not a function
```

因为在 success 回调函数中的 this 的指向发生了改变，不再指向 Vue 实例。
确保在回调函数中能够正确访问到 Vue 实例的方法，常用的两种解决方案：
1. 上面提到箭头函数

将 success 回调函数改为箭头函数，这样它会继承外层作用域的 this，即 Vue 实例。

```js
uni.showModal({
  //...
  success: (res) => {
    // 使用箭头函数
    if (res.confirm) {
        this.$request('/mini/shop/update', 'POST', {
        //...
      }).then(res => {
        //...
      });
    }
  }
});

```

2. 保存 this 引用

在进入 showModal 函数之前，将 this 保存在一个变量中，以便在回调函数中使用。

```js
let _this = this; // 保存 this 的引用
uni.showModal({
    //...
    success: function(res) {
        if (res.confirm) {
            _this.$request('/mini/wmsCheck/delete', 'POST', {
                //...
            }).then(res => {
                //...
            });
        } else if (res.cancel) {
            //...
        }
    }
});

```



## 什么是 this？

在 JavaScript 中，this 是一个特殊的关键字，用于指向当前执行上下文中的对象或函数。
[this 不能在执行期间被赋值，在每次函数被调用时 this 的值也可能会不同](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)，取决于代码的调用方式和上下文环境。

## 全局环境下的 this 指向

在全局环境下，this 指向全局对象（通常是 window 对象）。指在浏览器环境中，全局作用域下使用 this 可以直接访问 window 对象的属性和方法。
 
```js
console.log(this === window); // true
 ```

## 函数中的 this 指向

在函数内部，this 的指向可能有所不同，取决于函数的调用方式。在以下几种常见情况下，函数中的 this 会有不同的指向：

### 1.函数作为对象的方法调用：
当函数作为对象的方法被调用时，this 指向调用该方法的对象。

```js
const obj = {
    name: 'Alice',
    sayHello: function() {
        console.log('Hello, ' + this.name);
    }
};
obj.sayHello(); // 输出：Hello, Alice
```

### 2.函数作为普通函数调用：
当函数作为普通函数被调用时，this 指向全局对象（非严格模式）或 undefined（严格模式）。

```js
function sayHello() {
    console.log('Hello, ' + this.name);
}
sayHello(); // 在非严格模式下输出：Hello, undefined
```

需要注意的是，在 ECMAScript 5 的严格模式下，函数内部的 this 默认为 undefined，这有助于避免意外的全局变量污染。

### 3.构造函数中的 this 指向：
使用 new 关键字创建对象时，构造函数中的 this 指向新创建的实例对象。

```js
function Person(name) {
    this.name = name;
}
const person = new Person('Bob');
console.log(person.name); // 输出：Bob
```

## 箭头函数中的 this 指向

箭头函数是 ES6 中引入的新语法，它的 this 指向与所在上下文相同，而不是根据调用方式或上下文环境动态确定。

```js
const obj = {
    name: 'Alice',
    sayHello: function() {
        const arrowFunc = () => {
            console.log('Hello, ' + this.name);
        };
        arrowFunc();
    }
};
obj.sayHello(); // 输出：Hello, Alice
```

## bind()、call() 或 apply() 改变函数的执行上下文
```js
function sayHello() {
  console.log('Hello, ' + this.name);
}

const person = {
  name: 'Bob'
};

const boundFunction = sayHello.bind(person);
boundFunction(); // 输出：Hello, Bob

sayHello.call(person); // 输出：Hello, Bob

sayHello.apply(person); // 输出：Hello, Bob

```

## 总结：
在前端开发中，this 的指向取决于执行上下文，包括全局环境、函数调用方式以及箭头函数特殊性质等因素，合理运用 this 指向，使得代码更加灵活和易于维护。
