
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

    可以看出，`return` 之前的工作都白做了，最后返回 return 后面的对象。

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

创建一个新的空的对象。

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

3. 


#### 第三步

#### 第四步

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


## 参考文献

1. [前端面试题——自己实现new](https://zhuanlan.zhihu.com/p/84605717)
2. [详解 JS 中 new 调用函数原理](https://segmentfault.com/a/1190000015424508)
3. [Object.create()、new Object()和{}的区别](https://juejin.cn/post/6844903917835436045)