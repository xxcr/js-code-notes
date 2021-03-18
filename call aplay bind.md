# 

## apply、call

对于 apply、call 二者而言，作用完全一样，只是接受参数的方式不太一样。

### 作用

1. 在JavaScript中，`call()`和`apply()`都是为了改变函数运行时上下文而存在的，换句话说：就是为了改变函数体内部 this 的指向。
2. 传的第一个参数都是调佣函数时的this指向，通俗点就是想要使用前面的方法的对象。

```js
function Person () {}
 
Person.prototype = {
    name: "xuxu",
    say: function () {
        console.log("My name is " + this.name)
    }
}
 
var p1 = new Person
p1.say()  // My name is red
```

构造函数，在原型上添加公共属性和方法，不多说，前面已经详细介绍过[原型和原型链](https://juejin.cn/post/6938228794150879268)。
但有个对象想要使用这个 say() 方法，那我们就可以通过`call()`或者`apply()` 来使用。

```js
p2 = {
  name: '22'
}

Person.prototype.say.call(p2)
Person.prototype.say.apply(p2)  // My name is 22
```

可以看出：say 方法的this指向被改成了p2，所以 `call()`和`apply()` 作用就是动态改变 this，当一个 object 没有某个方法（本栗子中p2没有say方法），但是其他的有（本栗子中Person.prototype 或者 p1 有say方法），我们可以借助 `call()`或者`apply()` 用其它对象的方法来操作。

### 区别

第二个参数的不同

```js
func.call(this, arg1, arg2);
func.apply(this, [arg1, arg2])
```

### 注意

`call()`和`apply()` 如果不使用第一个参数，方法的this会被绑定为全局对象。在严格模式下，this 的值将会是 undefined。

```js

var a = 'a'

function log() {
  console.log('a = ', this.a);
}

log.call()  // a = a

```

严格模式

```js

'use strict'

log.call()  // // Cannot read the property of 'a' of undefined

```



### 实例

因为第二个参数的不同，所以 `call()`和`apply()` 有一些适用条件。当你的参数是明确知道数量时用 call 。而不确定的时候用 apply，然后把参数 push 进数组传递进去。当参数数量不确定时，函数内部也可以通过 arguments 这个类数组来遍历所有的参数。

#### 1. 数组后面追加一个数组

    `Array.prototype.push.apply(array1, array2)`
  
    当然 ES6 提供更简洁的形式 `arr1.push(...arr2)`

#### 2. 获取数组中的最大值和最小值

    `Math.max.apply(null, array1)`
  
    ES6：`Math.max(...array1)`

#### 3. 将类数组转换为数组

    `Array.prototype.slice.call(arrayLike)`
  
    ES6：`Array.from(arguments)` 或者 `[...arguments]`

    > JavaScript 类数组对象的定义：
    >  1. 可以通过索引访问元素，并且拥有 length 属性；
    >  2. 没有数组的其他方法，例如 push ， forEach ， indexOf 等。
    > ```js
    >  let foo = {
    >      0: 'Java',
    >      1: 'Python',
    >      2: 'Scala',
    >      length: 3
    >  }
    > ```

    >  arguments 对象，还有像调用 getElementsByTagName , document.childNodes 之类的，它们返回NodeList对象都属于类数组对象。

#### 4. 判断数据类, 前面介绍过

    `Object.prototype.toString.call(obj)`

#### 5. 构造函数继承

    借用父类的构造函数来增强子类实例，等于是复制父类的实例属性给子类。

    ```js

    function Parent(name) {
        this.name = name // 实例基本属性 (该属性，强调私有，不共享)
        this.arr = [1] // (该属性，强调私有)
        this.say = function() { // 实例引用属性 (该属性，强调复用，需要共享)
          console.log('hello')
        }
    }
    function Child(name, like) {
      Parent.call(this, name)  // 核心
      this.like = like
    }
    let boy1 = new Child('小红', 'apple')
    let boy2 = new Child('小明', 'orange ')

    // 优点1：可以向父类构造函数传参数
    console.log(boy1.name, boy2.name) // 小红， 小明

    // 优点2：子类实例不共享父类构造函数的引用属性
    boy1.arr.push(2)
    console.log(boy1.arr,boy2.arr)// [1,2] [1]

    // 缺点1：方法不能复用,每次创建子类实例都要创建一遍方法
    console.log(boy1.say === boy2.say) // false (说明，boy1和boy2 的say方法是独立，不是共享的)

    // 缺点2：不能继承父类原型上的方法
    Parent.prototype.walk = function () {   // 在父类的原型对象上定义一个walk方法。
      console.log('我会走路')
    }
    boy1.walk  // undefined (说明实例，不能获得父类原型上的方法)

    ```

#### 6. 面试题

    定义一个 log 方法，让它可以代理 console.log 方法。
    也许你直接写出：

    ```js

    function log (msg)　{
      console.log(msg)
    }

    log(1)    //1
    log(1,2)    //1

    ```

    上面这种方法传入多个参数就失效了，这个时候就可以考虑使用 apply 或者 call，注意这里传入多少个参数是不确定的，所以使用apply是最好的：

    ```js

    function log (){
      console.log.apply(console, arguments)
    }

    log(1)    //1
    log(1,2)    //1 2

    ```

    当然也可以用ES6：
    ```js
    function log (){
      console.log(...arguments)
    }
    ```

### 代码实现 `call()`和`apply()` 

以`call()`为例，`apply()` 只是参数的不同而已

实现思路：利用谁调用函数，函数的 this 就指向谁这一特点来实现。

  -   把函数 this 指向传入的对象 context
  -   执行 fun 方法
  -   由于我们添加了 fun 方法 ，所以需要删除 fun

```js

Function.prototype.myCall = function () {
  if (typeof this !== 'function') throw 'caller must be a function'

  // 不传参指向全局
  let context = arguments[0] || window

  // 防止覆盖掉原有属性
  const fun = Symbol()
  context[fun] = this

  // 参数
  const args = [...arguments].slice(1)

  // 执行方法
  const res = context[fun](...args)

  // 删除 fun 属性
  Reflect.deleteProperty(context, 'fun') 

  return res
} 

```


## 参考文献
1. [Function.prototype.apply()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
2. [Function.prototype.call()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
3. [Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
4. [javascript 基础之 call, apply, bind](https://zhuanlan.zhihu.com/p/71553017)
6. [【优雅代码】深入浅出 妙用Javascript中apply、call、bind](https://www.cnblogs.com/coco1s/p/4833199.html)
7. [[译] JavaScript 中至关重要的 Apply, Call 和 Bind](https://hijiangtao.github.io/2017/05/07/Full-Usage-of-Apply-Call-and-Bind-in-JavaScript/)
8. [JS中的call、apply、bind方法详解](https://segmentfault.com/a/1190000018270750)
