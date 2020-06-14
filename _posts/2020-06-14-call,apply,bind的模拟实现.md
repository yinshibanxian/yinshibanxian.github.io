---
title: call,apply,bind的模拟实现
categories:
- JS基础
- 面试问题
---

### Call

一句话介绍`call`:

`call() 方法在使用一个指定的 this 值和若干个指定的参数值的前提下调用某个函数或方法。`

例如:

```
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

bar.call(foo); // 1
```

注意两点:

1. `call`改变了`this`的指向，指向了`foo`
2. `bar`函数执行了

那我们如何模拟实现以上两点效果呢？不妨试想当调用`call`时，将`foo`对象改造成如下形式:

```
var foo = {
    value: 1,
    bar: function() {
        console.log(this.value)
    }
};

foo.bar(); // 1
```

这样`bar`函数中的`this`就指向了`foo`，我们最后再从`foo`函数当中删除`bar`这个方法即可.

综上，我们模拟的步骤可以分为:

1. 将函数设置为要指向的对象的属性
2. 执行该函数
3. 删除该函数

#### 第一版

```
var value = 2;
const foo = {
    value: 1
}

function bar() {
   console.log(this.value);
}

Function.prototype.myCall = function(context) {
    // 获取调用call的函数
    context.fn = this;
    context.fn();
    delete context.fn;
}
bar.call(foo); // 1
```

#### 第二版(带参数)

call函数还能给定参数执行函数，例如:

```
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}

bar.call(foo, 'kevin', 18);
// kevin
// 18
// 1
```

但是传入的参数并不确定，因此我们可以从函数内置的`arguments`当中取值，取出第二个及第二个参数以后的参数，如下:

```
// 以上个例子为例，此时的arguments为：
// arguments = {
//      0: foo,
//      1: 'kevin',
//      2: 18,
//      length: 3
// }
// 因为arguments是类数组对象，所以可以用for循环
var args = [];
for(var i = 1, len = arguments.length; i < len; i++) {
    // 之所以push的是字符串是为了方便后面eval函数解析执行
    args.push('arguments[' + i + ']');
}

// 执行后 args为 ['arguments[1]',...,arrguments[arguments.length-1]];
```

完整的第二版代码:

```
var value = 2;
const foo = {
    value: 1
}

function bar(name,age) {
    console.log(name);
    console.log(age);
   console.log(this.value);
}

Function.prototype.myCall = function(context) {
    // 获取调用call的函数
    context.fn = this;
    const args = [];
    for(let i = 1;i < arguments.length;i ++) {
        args.push('arguments['+i+']');
    }
    eval('context.fn('+args+')');
    delete context.fn;
}
bar.call(foo,'Kevin',18); // Kevin,18,1
```

#### 第三版（带返回结果）

调用的函数可能会有返回结果，例如:

```
var value = 2;
const foo = {
    value: 1
}

function bar(name,age) {
   console.log(name);
   console.log(age);
   console.log(this.value);
   return {
       name:name,
       age:age,
       value: this.value
   }
}
```

考虑到返回结果的代码

```
var value = 2;
const foo = {
    value: 1
}

function bar(name,age) {
   console.log(name);
   console.log(age);
   console.log(this.value);
   return {
       name: name,
       age: age,
       value: this.value
   }
}

Function.prototype.myCall = function(context) {
    // 当传入对象为null时，指向全局对象
    context = context || window;
    // 获取调用call的函数
    context.fn = this;
    const args = [];
    for(let i = 1;i < arguments.length;i ++) {
        args.push('arguments['+i+']');
    }
    const result = eval('context.fn('+args+')');
    delete context.fn;
    return result;
}
console.log(bar.call(foo,'Kevin',18)); // Kevin,18,1,{ name: 'Kevin', age: 18, value: 1 }

```

#### 第四版(指向对象为空)

`this`参数可以传`null`，当为`null`时，视为指向window

```

var value = 1;

function bar() {
    console.log(this.value);
}

bar.call(null); // 1
```

因此，完善代码为

```
Function.prototype.myCall = function(context) {
    // 当传入对象为null时，指向全局对象
    context = context || window;
    // 获取调用call的函数
    context.fn = this;
    const args = [];
    for(let i = 1;i < arguments.length;i ++) {
        args.push('arguments['+i+']');
    }
    const result = eval('context.fn('+args+')');
    delete context.fn;
    return result;
}
```

**最终代码**

```
Function.prototype.myCall = function(context) {
    // 当传入对象为null时，指向全局对象
    context = context || window;
    // 获取调用call的函数
    context.fn = this;
    const args = [];
    for(let i = 1;i < arguments.length;i ++) {
        args.push('arguments['+i+']');
    }
    const result = eval('context.fn('+args+')');
    delete context.fn;
    return result;
}
```

### Apply

`apply`函数同样可以改变调用函数中`this`的指向，只是第二个参数是数组的形式。模拟`apply`函数

与`call`类似。

```
const foo = {
    value: 1
}

function bar(name,age) {
   console.log(name);
   console.log(age);
   console.log(this.value);
   return {
       name: name,
       age: age,
       value: this.value
   }
}

Function.prototype.myApply = function(context,arr) {
    context = context || window;
    context.fn = this;
    let result;
    if(!arr) {
        result = context.fn();
    } else {
        const args = [];
        for(let i = 0;i < arr.length;i ++) {
            args.push('arr['+i+']');
        }
        result = eval('context.fn('+args+')');
    }
    delete context.fn;
    return result;
}
console.log(bar.myApply(foo,['Kevin',18])); // Kevin,18,1,{ name: 'Kevin', age: 18, value: 1 }
```

### Bind

一句话介绍`bind`:

`bind() 方法会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数。(来自于 MDN )`

由此可以总结出`bind`函数的两个特点:

1. 返回一个函数
2. 可以传入参数

#### 返回函数的模拟实现

从第一个特点开始，我们举个例子:

```
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

// 返回了一个函数
var bindFoo = bar.bind(foo); 

bindFoo(); // 1
```

因此，我们只需要返回一个改变了`this`指向的函数即可，而改变`this`指向可以通过`call`函数或者`bind`函数实现.

```
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

Function.prototype.myBind = function(context) {
    // 保存调用函数
    const self = this;
    return function() {
        self.apply(context);
    }
}

const bound = bar.bind(foo);
bound();
```

#### 传参的模拟实现

接下来看第二点，可以传入参数。需要注意的是，在函数调用bind时以及执行bind返回的函数的时候，都可以传参。

例如:

```
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(this.value);
    console.log(name);
    console.log(age);

}

var bindFoo = bar.bind(foo, 'daisy');
bindFoo('18');
// 1
// daisy
// 18
```

可以看到,`bar`函数需要传入`name`，`age`两个参数，我们可以在bind的时候传入一个参数，

再在执行bind返回的函数的时候，再传入另一个参数。因此，我们需要合并两次传入的参数。

```
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(name);
    console.log(age);
    console.log(this.value);

}

Function.prototype.myBind = function(context) {
    // 保存调用bind的函数
    const self = this;
    // 保存bind的时候传入的第二个参数及以后的参数
    const args = Array.prototype.slice.call(arguments,1);
    return function() {
        // 保存执行bind返回的函数时传入的参数
        const bindArgs = Array.prototype.slice.call(arguments);
        // 合并两次传入的参数
        self.apply(context,args.concat(bindArgs));
    }
}

const bound = bar.bind(foo,'Kevin',18);
bound(); // Kevin,18,1
```

#### 构造函数的模拟实现

完成了以上两点，还有一个最难的部分。因为`bind`函数还有一个特点:

> 一个绑定函数也能使用new操作符创建对象：这种行为就像把原函数当成构造器。提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。

也就是说当`bind`返回的函数作为构造函数的时候，bind时指定的this值会失效，但是传入的参数依然生效。举个例子:

```
var value = 2;

var foo = {
    value: 1
};

function bar(name, age) {
    this.habit = 'shopping';
    console.log(this.value);
    console.log(name);
    console.log(age);
}



bar.prototype.friend = 'kevin';

var bindFoo = bar.bind(foo, 'daisy');

var obj = new bindFoo('18');
// undefined
// daisy
// 18
console.log(obj.habit);
console.log(obj.friend);
// shopping
// kevin
```

上述代码中，尽管在全局和`foo`中都声明了value值，最后依然返回了`undefined`，说明绑定的`this`失效了，

因为`new`的原因，此时的this已经指向了obj。

所以我们可以通过修改返回的函数的原型来实现:

```
var value = 2;

var foo = {
    value: 1
};

function bar(name, age) {
    this.habit = 'shopping';
    console.log(this.value);
    console.log(name);
    console.log(age);
}

bar.prototype.friend = 'kevin';

Function.prototype.myBind = function(context) {
    // 保存调用bind的函数
    const self = this;
    // 保存bind的时候传入的第二个参数及以后的参数
    const args = Array.prototype.slice.call(arguments,1);
    const fBound = function() {
        // 保存执行bind返回的函数时传入的参数
        const bindArgs = Array.prototype.slice.call(arguments);

       /** 当返回的fBound函数作为构造函数调用时,this指向实例,self指向绑定函数
        instanceof 用来判断一个函数(对象)的原型是否在另一个函数(对象)的原型链上
        这里fBound在返回的时候原型便指向了示例的原型，因此实例的原型在fBound的原型链上
        */
       
        /**
         * 当返回的fBound函数作为一般函数时，this指向全局对象,self指向绑定函数，此时结果为false
         * 则让this指向绑定的context
         */
       
        self.apply(this instanceof self? this : context,args.concat(bindArgs));
    }
    // 将返回的fBound函数原型作为绑定函数的原型，那么fBound作为构造函数创建的实例就可以继承绑定函数原型链上的方法
    fBound.prototype = this.prototype;
    return fBound;
}

const bound = bar.myBind(foo,'Cindy',18);
const obj = new bound();
// console.log(obj);
console.log(obj.habit);
console.log(obj.friend);
```

#### 构造函数的优化实现

在上述代码实现中，有这么一行代码：

`fBound.prototype = this.prototype;`

这样当我们修改fBound.prototype上的属性，如添加一个新属性，绑定函数(bar)原型上的属性也会被修改。

因此我们需要一个空对象进行中转:

```
Function.prototype.myBind = function(context) {
    // 保存调用bind的函数
    const self = this;
    // 保存bind的时候传入的第二个参数及以后的参数
    const args = Array.prototype.slice.call(arguments,1);
    // 空对象作为中转
    const fNOP = function() {};
    const fBound = function() {
        // 保存执行bind返回的函数时传入的参数
        const bindArgs = Array.prototype.slice.call(arguments);

       /** 当返回的fBound函数作为构造函数调用时,this指向实例,self指向绑定函数
        instanceof 用来判断一个函数(对象)的原型是否在另一个函数(对象)的原型链上
        这里fBound在返回的时候原型便指向了示例的原型，因此实例的原型在fBound的原型链上
        */
       
        /**
         * 当返回的fBound函数作为一般函数时，this指向全局对象,self指向绑定函数，此时结果为false
         * 则让this指向绑定的context
         */
       
        self.apply(this instanceof self? this : context,args.concat(bindArgs));
    }
    // 将返回的fBound函数原型作为绑定函数的原型，那么fBound作为构造函数创建的实例就可以继承绑定函数原型链上的方法
    FNOP.prototype = this.prototype;
    fBound.prototype = new FNOP();
    return fBound;
}
```

以上代码即为最终版的bind模拟实现。





