## bind()

### 作用

`bind()` 方法创建一个新的函数，在 `bind()` 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。

通过代码来看：

```js

// 将document的write方法赋值
let altwrite = document.write

altwrite('hellow') // Uncaught TypeError: Illegal invocation

```

很简单，this指向的问题，write方法的this指向的是 document对象，赋值给 altwrite，altwrite没有通过调用就直接执行，this的指向global或window对象。
如果想要altwrite 方法的this 也指向document 对象，也可以使用前面介绍过的 [cal() 和 apply()]() 

```js

// 将document的write方法赋值
let altwrite = document.write

altwrite.bind(document)("hello")

altwrite.call(document, "hello")

```

`bind()` 返回一个函数，将altwrite 中的this指向 document对象，然后传入参数执行；`cal()` 将altwrite 中的this指向 document对象，document 直接使用 altwrite方法传入参数，返回执行后的值。

### cal() 、 apply() 、bind() 的比较

#### 1. 相同点：

-   都是用来改变函数的this对象的指向的；

-   第一个参数都是this要指向的对象，也就是想指定的上下文；

-   都可以利用后续参数传参，其中 `cal()` 和 `bind()` 的传参一样。

-   不使用第一个参数，方法的this会被绑定为全局对象。在严格模式下，this 的值将会是 undefined。

#### 2. 区别：

    `bind()` 是返回对应函数，便于稍后调用；`cal()` 和 `apply()` 则是立即调用，返回执行函数后的值。

### 实例

使用`cal()` 的地方也可以使用 `bind()`方法，就像上面一样，但 `bind()` 返回的是函数，又有自己的使用地方。

#### 1. 绑定函数

1. bind()最简单的用法是创建一个函数，使这个函数不论怎么调用都有同样的this值。常见的错误就像上面的例子一样，将方法从对象中拿出来，然后调用，并且希望this指向原来的对象。

      ```js

      let n = 0

      let a = {
        n: 1,
        getN: function() { 
          console.log(this.n)
        }
      }

      let b = { n: 2 }

      a.getN() // 1

      let getNc = a.getN
      getNc() // 0       this指向全局对象

      let getNcc = a.getNc.bind(a)
      getNcc() // 1

      b.getNcc() // 1     this都指向bind的第一个参数

      ```

2. ES6之前，我们都会使用_this , that , self 等保存 `this`，在改变了上下文之后继续引用到它

      ```js

      let a = {
        n: 1,
        getN: function() { 
          let _this = this
          $('.xx').on('click', function (event) {
            console.log(_this.n)     //1
          })
        }
      }

      ```

     当然使用ES6的箭头函数可以避免，也是首推的方法，这里讲bind的用法。

     ```js

       $('.xx').on('click', function (event) {
          console.log(this.n)     //1
       }).bind(this))

     ```

#### 2. 配合 setTimeout

这个和上面第二种类似。一般情况下setTimeout()的this指向window或global对象。

```js

  function Person (name) {
    this.name = name
  }

  // 1秒后调用getInfo函数
  Person.prototype.print = function () {
    window.setTimeout(this.getInfo.bind(this), 1000)
  }

  Person.prototype.getInfo = function () {
    console.log('My name is' + this.name)
  }

```

#### 3. 偏函数


#### 4. 绑定函数作为构造函数

#### 5. 捷径

`bind()` 也可以为需要特定this值的函数创造捷径。

以类数组对象转换为的数组为例，当然用前面介绍过的ES6 的 拓展运算 是最好的

```js

Array.prototype.slice.call(arguments)

```

但如果有很多类数组对象，就需要写


### 代码实现




### 最后注意点

1. 使用 `bind()` 绑定一个函数，后续再次使用 `bind()` 绑定没有作用。最后执行函数 fn 时，this 始终时被指向第一次 `bind()` 时的 thisArg。

2. 使用 `bind()` 绑定函数 this 后，不能使用  `cal()` 和 `bind()`  改变函数的指向。

## 参考文献
1. [Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
2. [前端面试题——自己实现bind](https://zhuanlan.zhihu.com/p/85438296)
3. [Javascript中bind()方法的使用与实现](https://segmentfault.com/a/1190000002662251)
4. [javascript 基础之 call, apply, bind](https://zhuanlan.zhihu.com/p/71553017)
5. [JS中的call、apply、bind方法详解](https://segmentfault.com/a/1190000018270750)
