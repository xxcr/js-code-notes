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

#### 2. 区别：

-   `bind()` 是返回对应函数，便于稍后调用；`cal()` 和 `apply()` 则是立即调用，返回执行函数后的值。

-   `cal()` 和 `bind()` 不使用第一个参数，方法的this会被绑定为全局对象。在严格模式下，this 的值将会是 undefined。
   
    `bind()` 如果 bind 函数的参数列表为空，或者thisArg是null或undefined，执行作用域 的 this 将被视为新函数的 第一个参数。

-   `bind()` 使用 new 运算符构造绑定函数，则忽略第一个参数。

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

bind() 的另一个最简单的用法是使一个函数拥有预设的初始参数。
只要将这些参数（如果有的话）作为 bind() 的参数写在 this 后面。当绑定函数被调用时，这些参数会被插入到目标函数的参数列表的开始位置，传递给绑定函数的参数会跟在它们后面。

```js

function list() {
  return Array.prototype.slice.call(arguments);
}

function addArguments(arg1, arg2) {
    return arg1 + arg2
}

var list1 = list(1, 2, 3); // [1, 2, 3]

var result1 = addArguments(1, 2); // 3

// 创建一个函数，它拥有预设参数列表。
var leadingThirtysevenList = list.bind(null, 37);

// 创建一个函数，它拥有预设的第一个参数
var addThirtySeven = addArguments.bind(null, 37);

var list2 = leadingThirtysevenList();
// [37]

var list3 = leadingThirtysevenList(1, 2, 3);
// [37, 1, 2, 3]

var result2 = addThirtySeven(5);
// 37 + 5 = 42

var result3 = addThirtySeven(5, 10);
// 37 + 5 = 42 ，第二个参数被忽略

```

#### 4. 绑定函数作为构造函数

绑定函数自动适应于使用 new 操作符去构造一个由目标函数创建的新实例。
当一个绑定函数是用来构建一个值的，原来提供的 this 就会被忽略。不过提供的参数列表仍然会插入到构造函数调用时的参数列表之前。

什么意思？先看一个栗子：

```js

function Person (name, age) {
  this.name = name
  this.age = age
  
  this.getInfo = function () {
    console.log(this.name + this.age)
  }
}

let p1 = new Person('xuxu', 18)
p1.getInfo() // xuxu18

```

使用`bind()` :

```js

let obj = {
  age: 19
}

let newPerson = Person.bind(obj, 'x')
let p2 = new newPerson(5)
p2.getInfo() // x5

```

根据 `bind()` 的用法，newPerson 的 this 是指向obj，但使用new new运算符构造时，提供的this将被忽略，所以this指向了p2。


#### 5. 捷径

`bind()` 也可以为需要特定this值的函数创造捷径。

以类数组对象转换为的数组为例，当然用前面介绍过的ES6 的 拓展运算 是最好的

```js

Array.prototype.slice.call(arguments)

```

但如果有很多类数组对象，就需要写很多的个这样的表达式，所以可以使用 `bind()` 

```js

let slice = Function.prototype.call.bind(Array.prototype.slice)

slice(arguments)

```


### 代码实现 （ES6以前）

1. 先说好，这里使用ES6以前的方法实现，使用ES6可以简化一些操作。

2. 使用 `call()` 和 `apply()`，如果要求不用call，自己实现个call，然后替换。

举个栗子：

```js

let a = b.bind(obj, '1', '2')

a('3')

```

#### 第一步

改变this指向，返回个函数稍后执行。

1. arguments中第一个元素就是我们调用bind时候传入的第一个参数obj，使用apply这个函数，调用了a，并把a中的this指向了obj。

    ```js
    
    Function.prototype.myBind = function () {
        
      return this.apply(arguments[0])
    }
    
    ```

2. 由于bind函数并不是立即执行，而是要返回一个函数，所以包装在一个函数当中进行返回。（ES6 可以使用箭头函数）

    ```js
    
    Function.prototype.myBind = function () {
      let self = this
    
      return function () {
        return self.apply(arguments[0])
      }
    }
    
    ```

3. 最后再判断一下a是否是 function，就完美了

    ```js
    
    Function.prototype.myBind = function () {
      if (typeof this !== 'function') {
        throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable')
      } 
    
      let self = this
    
      return function () {
        return self.apply(arguments[0])
      }
    }
    
    ```

这样就完美的完成了 __第一步：改变this指向，返回个函数__

#### 第二步

调用 `bind()` 时，除了传需要绑定this值的对象外，还可能传入其他参数 '1', '2'。将arguments类数组对象截取除obj 转换成数组（ES6 可以使用 [...arguments]），得到的args传入apply函数（apply参数时数组）。

```js

Function.prototype.myBind = function () {
  if (typeof this !== 'function') {
    throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable')
  } 

  let self = this
  let args = Array.prototype.slice.call(arguments, 1)

  return function () {
    return self.apply(arguments[0], args)
  }
}

```

#### 第三步

在使用a函数的时候，也可能传入参数，所以需要将传入的参数合并到 `apply()` 的第二个参数中。

我们使用a函数的时候，实际上是执行return的那个函数，即

```js

return function () {
  return self.apply(arguments[0], args)
}

```

所以，只需要将这个函数的 arguments 拼在 args 的后面。（ES6更简洁）

```js

Function.prototype.myBind = function () {
  if (typeof this !== 'function') {
    throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable')
  } 

  let self = this
  let thatArgs = arguments[0]
  let args = Array.prototype.slice.call(arguments, 1)

  return function () {
    Array.prototype.push.apply(args, Array.prototype.slice.call(arguments))
    return self.apply(thatArgs, args)
  }
}

```

到这一步基本已经实现上面栗子中的bind。

#### 第四步

绑定函数自动适应于使用 new 操作符去构造一个由目标函数创建的新实例。当一个绑定函数是用来构建一个值的，原来提供的 this 就会被忽略。不过提供的参数列表仍然会插入到构造函数调用时的参数列表之前。

这一步主要实现这个功能的，意思是使用 new 构造一个由目标函数创建的新实例

```js

let newPerson = Person.bind(obj, 'x')

console.log(newPerson) // f Person() 返回Person函数

let p2 = new newPerson()

``` 
忽略 `bind()` 中的obj，然后this指向为创建的实例p2。

1. 第三步实现的返回的函数叫 fBound 吧，可以看到，返回的 fBound 函数还只是调用`bind()` 方法的那个函数，即上面的 `Person` ，在执行的时候才绑定this值 和 参数。根据 new 的定义，在 new 的时候，this 值指向创建的实例，所以返回的 fBound 被 new 的时候，this是指向实例的，所以我们判断 fBound 是否在this的原型链上，在则说明是通过new 调用了，返回的`Person` 的this 指向实列，否则指向 obj。

    在new关键字调用下，p2 “继承”自 Person.prototype 的实例，所以，我们把 fBound 的 prototype 修改为 Person 的 prototype。

    ```js
    
    Function.prototype.myBind = function () {
      if (typeof this !== 'function') {
        throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable')
      } 

      let self = this
      let thatArgs = arguments[0]
      let args = Array.prototype.slice.call(arguments, 1)

      let fBound  = function () {
        Array.prototype.push.apply(args, Array.prototype.slice.call(arguments))
        return self.apply( this instanceof fBound ? this : thatArg, args)
      }

      fBound.prototype = self.prototype

      return fBound
    }

    ```

2. 上面代码有个问题：fBound 的原型 就是 绑定函数即 Person 的 原型，但我们改了fBound.prototype 即 `newPerson.prototype = {}` ，这样Person的原型也被修改了。因此，我们需要一个中间变量fNOP，让它等于一个空函数，通过fNOP来维护原型关系，并让fBound.prototype 与 Person.prototype 不再指向同一个原型函数：

    ```js

    Function.prototype.myBind = function () {
      if (typeof this !== 'function') {
        throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable')
      } 

      let self = this
      let thatArgs = arguments[0]
      let args = Array.prototype.slice.call(arguments, 1)

      let fBound  = function () {
        Array.prototype.push.apply(args, Array.prototype.slice.call(arguments))
        return self.apply( this instanceof fBound ? this : thatArg, args)
      }

      let fNOP = function () {}

      if (self.prototype) {
        fNOP.prototype = self.prototype
      }

      fBound.prototype = new fNOP()

      return fBound
    }

    ```

    fNOP和 Person 使用同一个 prototype，而 fBound.prototype 是fNOP的一个实例，而这个实例的 \_\_proto\_\_ 才指向的是 Person.prototype。因此，直接修改 fBound.prototype并不会修改 Person的prototype。


#### 结尾
以上实现 `bind()` 与实际的算法还有许多其他的不同。借用[Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) 上面的话来说，尽管可能还有其他不同之处，但已经没有必要再多列举。


### 最后注意点

1. 使用 `bind()` 绑定一个函数，后续再次使用 `bind()` 绑定没有作用。最后执行函数 fn 时，this 始终时被指向第一次 `bind()` 时的 thisArg。

2. 使用 `bind()` 绑定函数 this 后，不能使用  `cal()` 和 `bind()`  改变函数的指向。

## 参考文献
1. [Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
2. [前端面试题——自己实现bind](https://zhuanlan.zhihu.com/p/85438296)
3. [Javascript中bind()方法的使用与实现](https://segmentfault.com/a/1190000002662251)
4. [javascript 基础之 call, apply, bind](https://zhuanlan.zhihu.com/p/71553017)
5. [JS中的call、apply、bind方法详解](https://segmentfault.com/a/1190000018270750)
