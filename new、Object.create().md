
## 前言

这里特别详细的介绍了`new` 运算符和 `Object.create()`，以及他们是如何实现的，最后还简单介绍下 __Object.create()、new Object()和{}的区别__

## new运算符

`new` 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。

### new的作用

#### 先来看三个栗子：

1. 常见的 `new` 的使用

    ```js

    function Test (name) {
      this.name = name
    }
    Test.prototype.sayName = function () {
        console.log(this.name)
    }

    const t = new Test('xx')
    console.log(t.name) // 'xx'
    t.sayName() // 'xx'
    
    ```

    使用构造函数创建对象的过程，不多说，大家都懂。那如果构造函数有 `return` 值呢？

2. `return` 对象类型数据

    ```js

    function Test (name) {
      this.name = name

      return {a: '啊啊啊'}
    }

    const t = new Test('xx')
    console.log(t) // '{a: '啊啊啊'}'

    ```

    可以看出，`return` 之前的工作都白做了，最后返回 `return` 后面的对象。

3. `return` 基本类型数据

    ```js

    function Test (name) {
      this.name = name

      return 1
    }

    const t = new Test('xx')
    console.log(t) // '{name: "xx"}'

    ```

    和没有 `return` 时效果一样。

我们可以得出一个结论：__构造函数如果返回值为对象，那么这个返回值会被正常使用，否则，就会默认返回新对象__

> 这例子告诉了我们一点：构造函数尽量不要返回值。因为返回原始值不会生效，返回对象会导致 `new` 操作符没有作用。

#### 作用

1. 创建一个新的空的对象。

2. 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）。

3. 执行构造函数中的代码（为这个新对象添加属性）。

4. 如果这个函数有返回值且返回值是对象，则返回；否则，就会默认返回新对象。

> 箭头函数中没有 [[Construct]] 方法，不能使用 `new` 调用，会报错。

### new的实现

根据上面那四个的作用，分成四步来实现一个 `new`

这个函数当中，第一个参数就是构造函数，第二个参数开始，是构造函数中的参数。

#### 第一步

__创建一个新的空的对象__。

```js

function myNew () {
  let obj = {}
}

```

#### 第二步

__将构造函数的作用域赋给新对象__ ，就是给这个新对象构造原型链，链接到构造函数的原型对象上，从而新对象就可以访问构造函数中的属性和方法。

1. 构造函数就是我们传入的第一个参数。由于`arguments` 是类数组，我们不能直接使用 `shift` 方法，我们可以使用 `call` 来调用 `Array` 上的 `shift` 方法，获取 `constr`：

    ```js

    let constr = Array.prototype.shift.call(arguments)

    ```

2. 把obj的原型指向构造函数的原型对象上：

    ```js

    obj.__proto__ = constr.prototype

    ```

    或者可以使用`setPrototypeOf`方法（设置一个指定的对象的原型 ( 即, 内部[[Prototype]]属性）到另一个对象或  `null`）：

    ```js

    Object.setPrototypeOf(obj, constr.prototype)

    ```

3. 简化：用 `Object.create()` 来简化上面两步：

    ```js

    let obj = obj = Object.create(constr.prototype)

    ```

    使用 `Object.create(object, objectProps)` 这种方法，第一个参数就是要创建的对象的原型


#### 第三步

__执行构造函数中的代码（为这个新对象添加属性）__。使用`apply` 实现：

```js

constr.apply(obj, arguments)

```

#### 第四步

__如果这个函数有返回值且返回值是对象，则返回；否则，就会默认返回新对象__

1. 首先，需要获取这个返回值：

    ```js

    let res = constr.apply(obj, arguments)

    ```

2. 从上面得出：`new` 关键字，如果返回的是 `undefined，null` 以及基本类型的时候，都会返回新的对象；而只有返回对象的时候，才会返回构造函数的返回值。

    所以：判断 `res` 是不是 `object` 类型，如果是 `object` 类型，那么就返回 `res`，否则，返回 `obj`。

    ```js

    return res instanceof Object ? res : obj

    ```

    > 使用 `res instanceof Object` 就能判断 `res` 是否为对象类型。
    > 以前都是用 `typeOf()` 蠢办法：`target !== null && (typeof target === 'object' || typeof target === 'function')`。


#### 最终代码

```js

function myNew () {
  let constr = Array.prototype.shift.call(arguments)
  let obj = Object.create(constr.prototype)
  let res = constr.apply(obj, arguments)
  return res instanceof Object ? res : obj
}

```

测试：`myNew(Test, 'xuxu')` 和上面一致。


## Object.create()

`Object.create()` 方法创建一个新对象，使用现有的对象来提供新创建的对象的 `__proto__`

### 语法

```js

Object.create(proto, [propertiesObject])

```

1. `proto` 必填参数，是新对象的原型对象。

   > 注意，如果这个参数是null，那新对象就彻彻底底是个空对象，没有继承 `Object.prototype` 上的 任何属性和方法，如 `hasOwnProperty()、toString()` 等。

2. `propertiesObject` 是可选参数。要定义其可枚举属性或修改的属性描述符的对象。对象中存在的属性描述符主要有两种：__数据描述符__ 和 __访问器描述符__ 。

    > 更多的请移步：[关于Object.defineProperty() 和 Object.defineProperties()](https://juejin.cn/post/6861089494158802957)

    ```js

    let xx = Object.create({a: 1}, {
      b: {
          value: 2,
          writable: false,
          configurable: true
      }
    })

    console.log(xx) // {b: 2}
    console.log(xx.__proto__) // {a: 1}  新对象xx的__proto__指向{a: 1}
    
    xx.b = 77;
    // throws an error in strict mode
    
    console.log(xx.b);
    // expected output: 2

    ```

### 用途

最大的用途就是实现js的继承：__原型式继承__ 和 __寄生组合继承__

> 更多的请移步以前的文章：[JavaScript的继承详解（ES5和ES6）--每天进步一点点](https://juejin.cn/post/6960310756885659661)

### 实现

思路很简单：创建一个新对象，将传入的第一个参数作为这个对象的原型，如果发现传递了第二个参数，通过 `Object.defineProperties` 为创建的对象设置 `key、value`，最后返回创建的对象即可。

1. 我自己的实现，感觉没什么问题：（todo：待纠正）

    ```js

    Object.myCreate = function (proto, propertyObject = undefined) {
      let obj = {}
      obj.__proto__ = proto
      if (propertyObject !== undefined) {
        Object.defineProperties(obj, propertyObject)
      }
      return obj
    }

    ```

2. 源码：定义一个空的构造函数，然后指定构造函数的原型对象，通过 `new` 运算符创建一个空对象。

    ```js

    Object.myCreate = function (proto, propertyObject = undefined) {
      if (propertyObject === null) {
        // 这里没有判断propertyObject是否是原始包装对象
        throw 'TypeError'
      } else {
        function F() {}
        F.prototype = proto
        const obj = new F()
        if (propertyObject !== undefined) {
          Object.defineProperties(obj, propertyObject)
        }
        if (proto === null) {
          // 创建一个没有原型对象的对象，Object.create(null)
          obj.__proto__ = null
        }
        return obj
      }
    }

    ```

    > 个人理解：`if (proto === null)` 这个没有必要，因为，如果 `proto = null`，上面这句 `F.prototype = proto` ，`obj` 的原型对象已经是 `null` 了。

## Object.create()、new Object()和{}的区别
 
1. `new Object()` 和 `{}` 没有什么区别，创建的新对象的 `__proto__` 都指向 `Object.prototype`。

2. `Object.create(proto, [propertiesObject])` 创建的对象的原型取决于 `proto` ， `proto` 为 `null` ，新对象是空对象，没有原型，不继承任何对象； `proto` 为指定对象，新对象的原型指向指定对象，继承指定对象

## 参考文献

1. [前端面试题——自己实现new](https://zhuanlan.zhihu.com/p/84605717)
2. [详解 JS 中 new 调用函数原理](https://segmentfault.com/a/1190000015424508)
3. [Object.create()、new Object()和{}的区别](https://juejin.cn/post/6844903917835436045)
4. [Object.create 实现原理](https://juejin.cn/post/6844904174983872519)