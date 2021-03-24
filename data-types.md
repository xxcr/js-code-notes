
## js数据类型

1. JavaScript 是弱类型或者说动态语言，变量类型随程序运行而改变。
2. 最新的 ECMAScript 标准定义了 8 种数据类型:
-   基本类型：Boolean、Null、Undefined、Number、BigInt、String、Symbol 
-   引用类型：Object。细分的话有：Object 、Array 、Date 、RegExp 、Function 、[Maps、 Sets、 WeakMaps、 WeakSets](https://github.com/xxcr/js-code-notes/blob/main/map%20set%20weakMap.md)
约定：基本数据类型与原始数据类型等意。

#### 详细
1. Symbol： 是ES6中引入的一种原始数据类型，表示独一无二的值。
2. BigInt：是 ES2020 引入的一种新的数据类型，用来解决 JavaScript中数字只能到 53 个二进制位，JavaScript 所有数字都保存成 64 位浮点数，大于这个范围的整数，无法精确表示的问题。
3. Number:

-   遵循 IEEE 754 标准 通过64位来表示一个数字（整数和浮点数）
-   详细参考：[Number精度](https://github.com/xxcr/js-code-notes/blob/main/Number%E7%B2%BE%E5%BA%A6.md)
    
#### 基本类型和引用类型区别
1. 存储的地址：
-   基本类型：存储在栈中，直接存储在变量访问的位置，便于迅速查寻变量的值。
-   引用类型：栈和堆中，栈中变量处存储的是一个指针，指向存储在堆内存中的对象的地址。

>这是因为：引用值的大小会改变，所以不能把它放在栈中，否则会降低变量查寻的速度。相反，放在变量的栈空间中的值是该对象存储在堆中的地址。地址的大小是固定的，所以把它存储在栈中对变量性能无任何负面影响。

2. 访问：
-   基本类型可以直接访问到，引用类型需要根据引用地址去获取。

3. 赋值：
-   基本类型：开辟一个新的栈内存地址，把值复制一份，两个相互不影响。
-   引用类型：栈中的指针复制出一个，但两个地址指向的是堆内存中的同一个对象，其中一个改变，另一个也会改变。

#### === 和 == 区别
   简单来说，==先转换类型再进行===比较，===先判断类型，如果不是同一类型直接为false。

##### === 简单不多说
1. 数值：如果其中至少一个是NaN，那么[不相等]。（判断一个值是否是NaN，只能用isNaN()来判断） 
2. 对象：引用同一个对象则相等
3. 两个都为null 或者 都为undefined ，相等

##### == 类型不同转换规则
1. null == undefined
2. 字符串 == 数值， 字符串转换成数值再比较
3. true转为 1， false 转为 0，再进行比较。'0' == false
4. 如果一个是对象，另一个是数值或字符串，把对象转换成基础类型的值再比较。对象转换成基础类型，利用它的toString或者valueOf方法。js核心内置类，会尝试valueOf先于toString；例外的是Date，Date利用的是toString转换。
5. 其他任何组合，不相等


## 判断
### 1. typeof

  -   操作符而不是函数，其右侧跟一个一元表达式。
  -   ```js
      typeof(Symbol())  // symbol
      typeof(null) // object
      function a() {}
      typeof a // function
      var date = new Date()
      typeof date      // object
      ```
      typeof可以识别除null以外的标准类型（null会识别为object），不能识别除Function以外的具体对象类型。
  -   原理：不同的对象在底层都表示为二进制，在Javascript中二进制前（低）三位存储其类型信息。
      > 000: 对象
        010: 浮点数
        100：字符串
        110：布尔
        1：整数
        null 全是0
      所以：typeof(null) 为对象。
  -   使用typeof来获取一个变量是否存在。如 : if(typeof a!="undefined"){}，而不要去使用if(a)，因为如果a不存在（未声明）则会出错。

  -   >注意：var a; typeof a; // 'underfined'
            typeof b; // 'underfined'

### 2. instanceof 
    
  -   instanceof 操作符主要用来检查构造函数的原型是否在对象的原型链上。可以在继承关系中用来判断一个实例是否属于它的父类型。
  -   语法： object instanceof constructor 
      参数：object（要检测的对象.）constructor（某个构造函数）
      描述：instanceof 运算符用来检测 constructor.prototype 是否存在于参数 object 的原型链上。
  -   instanceof 测试的 object 是指 js 语法中的 object，不是指 dom 模型对象。
      window instanceof Object // false。但：typeof(window) // object
  -    instanceof 识别所有的对象类型，但不能判别原始类型。
       ```js
        12 instanceof Number  // false
        '22' instanceof String  // false
        true instanceof Boolean // false
        null instanceof Object // false
        undefined instanceof Object // false

        var bool2 = new Boolean()
        bool2 instanceof Boolean // true

        function Person(){}
        var per = new Person()
        per instanceof Person // true

        [] instanceof Array // true
        [] instanceof Object // true
       ```
  -   基本类型如果不是用new声明的则也测试不出来，对于是使用new声明的类型，它还可以检测出多层继承关系。
  
  #### 代码实现
  思路：循环查找，判断左边对象的原型是否等于右边构造函数的原型，不等则继续向上查找，最终为null，则不存在
  ```js
  instanceof (left, right) => {
    if (typeof left !== 'object' || left === null) return false // 基础数据类型
    let proto  = left.__proto__
    while (true) {
      if (proto === null) return false
      if (proto === right.prototype) return false
      proto = proto.__proto__
    }
  }
  ```
  > 获取对象的原型两种方法：obj.__ proto __  和 Object.getPrototypeOf(obj)

[原型和原型链](https://github.com/xxcr/js-code-notes/blob/main/%E5%8E%9F%E5%9E%8B%E5%92%8C%E5%8E%9F%E5%9E%8B%E9%93%BE.md)

### 3. constructor
  -   constructor属性指向创建当前对象的构造函数
      ```js
      ''.constructor == String  // true 
      true.constructor == Boolean // true
      [].constructor == Array // true

      document.constructor == HTMLDocument // true
      window.constructor == Window // true

      new Number(1).constructor == Number // true
      new Function().constructor == Function // true
      ```
  -   null 和 undefined 没有constructor，不能判断，原因在下面。
  -   函数的 constructor 是不稳定的，当开发者重写 prototype 后，原有的 constructor 引用会丢失，constructor 会默认Object。

       ```js
        function Foo() {}
        Foo.prototype = {}
        var foo = new Foo()
        foo.constructor === Object // true，可以看出不是Foo了
      ```

  ##### constructor 是不稳定的原因：
   1. 起源：constructor属性是在原型里
       ```js
       function test() {}
       var obj = new test();
       console.log(obj.hasOwnProperty('constructor')); //false
       console.log(obj.__proto__.hasOwnProperty('constructor')); //true
       ```
   2. 所以上面：
   -   foo没有constructor属性，往原型Foo.prototype上找。
   -   Foo.prototype本来是有constructor属性的，是Function，但被重新定义为{}。
   -   找不到constructor，接着往原型链上查找。（{}是由Object构造函数生成的，它的原型是Object.prototype）找到了Object.prototype，含有属性constructor，值为Object
     
### 4. Object.prototype.toString.call()
  -  > 原理(摘自高级程序设计3)：在任何值上调用 Object 原生的 toString() 方法，都会返回一个 [object NativeConstructorName] 格式的字符串。每个类在内部都有一个 [[Class]] 属性，这个属性中就指定了上述字符串中的构造函数名。

      但是它不能检测非原生构造函数的构造函数名。

  -   例：    
       ```js
      Object.prototype.toString.call(1);  '[object Number]'
      Object.prototype.toString.call(NaN); '[object Number]'
      Object.prototype.toString.call('1'); '[object String]'
      Object.prototype.toString.call(true); '[object Boolean]'
      Object.prototype.toString.call(undefined); '[object Undefined]'
      Object.prototype.toString.call(null); '[object Null]'  
      Object.prototype.toString.call(Symbol());'[object Symbol]' 
      Object.prototype.toString.call(foo);  '[object Function]'  
      Object.prototype.toString.call([1,2,3]); '[object Array]'   
      Object.prototype.toString.call({});'[object Object]'
      ```
 -   call()方法可以改变this的指向，那么把Object.prototype.toString()方法指向不同的数据类型上面，返回不同的结果

### 5. 其他单一判断的
 -   Array.isArray()
     用于确定传递的值是否是一个Array。如果对象是Array，则返回true，否则为false。
 -   isNaN()和Number.isNaN
     用来确定一个值是否为 NaN。
     >isNaN(undefined); // true   
     isNaN({}); // true   
     isNaN()只要不是number就会返回 true。所以使用Number.isNaN()

 -   判断是否是 DOM 元素，利用的是 DOM 对象特有的 nodeType 属性
     ```js
     function (obj){
       return !!(obj && obj.nodeType === 1)
     }
     ```

 -   判断是否是 arguments 对象，使用：
     Object.prototype.toString.call(obj)   // '[object Arguments]'
     > 低版本的浏览器不支持, 他们返回的是 [object Object], 所以需要兼容：
     原理是通过对象的 hasOwnProperty 方法来判断对象是否拥有 callee 属性从而判断是不是 arguments 对象
      ```js
      function (obj){
        return Object.prototype.toString.call(obj) === '[object Arguments]' || (obj != null && Object.hasOwnProperty.call(obj, 'callee'))
      }
      ```

## 参考文献
1. [JavaScript中constructor属性](https://segmentfault.com/a/1190000013245739)
2. [彻底弄清JS数据类型判断](https://zhuanlan.zhihu.com/p/129642585)
