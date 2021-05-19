
## 前言

本文详细的介绍了 __赋值、浅拷贝、深拷贝__ ，其中涉及到一些知识点也做了介绍：[js数据类型及判断](https://github.com/xxcr/js-code-notes/blob/main/data-types.md)、[对象遍历](https://github.com/xxcr/js-code-notes/blob/main/%E9%81%8D%E5%8E%86%E5%AF%B9%E8%B1%A1.md)、WeakMap、JS数组遍历的几种方式性能对比 等等。

## 赋值

1. a = b

2. 基本数据类型：两个变量不影响

3. 引用数据类型：只复制地址，两个变量指向同一个地址，a和b同一个对象

[js数据类型及判断](https://github.com/xxcr/js-code-notes/blob/main/data-types.md)



## 浅拷贝

1. 与赋值不同，拷贝一层对象

2. 如果属性是基本类型，拷贝的就是基本类型的值，如果属性是引用类型，拷贝的就是内存地址。

3. 一般的数组和对象的赋值和方法都是浅拷贝，例如：

	-   <font color="#dd0000">Object.assign(a, b)</font>：把所有可枚举属性从一个或多个对象复制到目标对象，返回目标对象。
	-   array.concat()、Array.prototype.slice()
	-   [...a]

4. 代码实现：

    ```js

    const shallowClone = (target) => {
      if (typeof target === 'object' && target !== null) { // 对象，数组，但不为null
        const cloneTarget = Array.isArray(target) ? [] : {}

        for (item in target) { 
          if (target.hasOwnProperty(item)) { // 是否是自身属性（非继承）
            cloneTarget[item] = target[item]
          }
        }
        return cloneTarget
      } else {
        return target // 基础类型直接返回
      }
    }

    ```

### 判断对象属性是否存在

1. hasOwnProperty() 只能检测自有属性。

2. in 可以检测自有属性和继承属性。

3. 使用!==检测，使用！==需要注意对象的属性值不能设置为undefined
   注意必须是！==，而不是！= 因为！=不区分undefined和null


### 和赋值区别

```js

let a = {
  name: 1,
  address: {
    city: 2,
    home: 3
  }
}

let b = a // 赋值
let c = shallowClone(a) // 浅拷贝

a.name = 4
a.address.city = 5

console.log(a) // {name: 4, address: {city: 5, home: 3}}
console.log(b) // {name: 4, address: {city: 5, home: 3}} // 复制
console.log(c) // {name: 1, address: {city: 5, home: 3}} // 浅拷贝

```

## 深拷贝

1. 深拷贝是将一个对象从内存中完整的拷贝一份出来,从堆内存中开辟一个新的区域存放新对象。
完全复制出一个对象，连地址也拷贝，两个不会相互影响。嵌套对象也完全拷贝。

2. 乞丐版：

    ```js

    JSON.parse(JSON.stringify(obj))

    ```

    ```js

    jQuery.extend()

    ```

3. 存在的问题：
	-   无法解决循环引用，会报错
	-   无法拷贝函数、RegExp、Date、Set、Map等特殊的对象。
	-   会忽略undefined、symbol。
	-   不能序列化函数

4. 工具库：lodash 的 cloneDeep 方法

### 实现

参照  https://juejin.cn/post/6902060047388377095 和 https://segmentfault.com/a/1190000020255831

#### 基础版本：

只考虑普通对象和数组

```js

const cloneDeep = (target) => {
  
  if (typeof target === 'object' && target !== null) { // 对象，数组，但不为null
    const cloneTarget = Array.isArray(target) ? [] : {}

    for (item in target) { 
      if (target.hasOwnProperty(item)) { // 是否是自身属性（非继承）
        cloneTarget[item] = cloneDeep(target[item])
      }
    }
    return cloneTarget
  } else {
    return target // 基础类型直接返回
  }
}

```

#### 循环引用：

1. 对象的属性间接或直接的引用了自身，例如：

    ```js

    const target = {
      a: 1,
      b: {
          bb: 2
      },
    }
    target.c = target

    ```

2. 解决循环引用问题：我们可以额外开辟一个存储空间，来存储当前对象和拷贝对象的对应关系，当需要拷贝当前对象时，先去存储空间中找，有没有拷贝过这个对象，如果有的话直接返回，如果没有的话继续拷贝，这样就巧妙化解的循环引用的问题。

这个存储空间，需要可以存储key-value形式的数据，且key可以是一个引用类型，自然而然想到 `Map` 这种数据结构。

3. 思路：

    -   检查map中有无克隆过的对象
    -   有 - 直接返回
    -   没有 - 将当前对象作为key，克隆对象作为value进行存储
    -   继续克隆

4. 所以代码可以这样写：

    ```js
    
    const cloneDeep = (target, map = new Map()) => {
      
      if (typeof target === 'object' && target !== null) { // 对象，数组，但不为null
        const cloneTarget = Array.isArray(target) ? [] : {}

        // 循环引用
        if (map.get(target)) return map.get(target)
        map.set(target, cloneTarget)

        for (item in target) { 
          if (target.hasOwnProperty(item)) { // 是否是自身属性（非继承）
            cloneTarget[item] = cloneDeep(target[item])
          }
        }
        return cloneTarget
      } else {
        return target // 基础类型直接返回
      }
    }

    ```
5. 优化：使用 `WeakMap` 提代 `Map` 

##### WeakMap

什么是`WeakMap`：

> `WeakMap` 对象是一组键/值对的集合，其中的键是弱引用的。其键必须是对象，而值可以是任意的。

什么是弱引用：

> 在计算机程序设计中，弱引用与强引用相对，是指不能确保其引用的对象不会被垃圾回收器回收的引用。 一个对象若只被弱引用所引用，则被认为是不可访问（或弱可访问）的，并因此可能在任何时刻被回收。

我们默认创建一个对象：`const obj = {}`，就默认创建了一个强引用的对象，我们只有手动将 `obj = null`，它才会被垃圾回收机制进行回收，如果是弱引用对象，垃圾回收机制会自动帮我们回收。

举个例子：

```js

let obj1 = { a: 1 }
let obj2 = new Map()

obj2.set(obj, 2)

obj1 = null // 将obj1进行释放

```

上面虽然我们将`obj1` 进行释放，但 `obj1` 和 `obj2` 之间存在强引用惯性系，所以这部分内存依然无法被释放。

把上面的 `map` 换成 `WeakMap`，target和obj存在的就是弱引用关系，当下一次垃圾回收机制执行时，这块内存就会被释放掉。

所以：

设想一下，如果我们要拷贝的对象非常庞大时，使用 `Map` 会对内存造成非常大的额外消耗，而且我们需要手动清除 `Map` 的属性才能释放这块内存，而 `WeakMap` 会帮我们巧妙化解这个问题。

#### 其他数据类型

1. 上面只考虑普通对象和数组，但还有很多数据类型。所以我们需要分情况讨论：

    -   基本数据类型直接返回。
    -   `set、map` 等这些都是 `可持续遍历` ，函数、正则等是不可持续遍历的，都需要单独进行克隆。

2. 我们不能再像上面那样 `const cloneTarget = Array.isArray(target) ? [] : {}` 获取它们的初始化数据。可以通过拿到 `constructor` 的方式来通用的获取。

    ```js
    
    const cloneTarget = new target.constructor()

    ```

    `const target = {}` 就是 `const target = new Object()` 的语法糖。
    这种方法还有一个好处：因为我们还使用了原对象的构造方法，所以它可以保留对象原型上的数据，如果直接使用普通的{}，那么原型必然是丢失了的。

3. 获取准确的引用类型：`toString()` 方法。

> 每一个引用类型都有 `toString()` 方法，默认情况下，`toString()` 方法被每个 `Object` 对象继承。如果此方法在自定义对象中未被覆盖，`toString()` 返回 `"[object type]"`，其中type是对象的类型。

注意：上面提到了如果此方法在自定义对象中未被覆盖，`toString()` 才会达到预想的效果，事实上，大部分引用类型比如 `Array、Date、RegExp` 等都重写了 `toString()` 方法。

所以：调用 `Object` 原型上未被覆盖的 `toString()` 方法，使用 `call` 来改变 `this` 指向，就能获取准确的引用类型。

   `Object.prototype.toString.call(target)`

##### 1. 判断引用类型

通过 `typeof` 准确的判断是否是引用类型：

   ```js
   
   if (target === null || (typeof target !== 'object' && typeof target !== 'function')) {
     return target
   }
   
   ```

##### 2. 可继续遍历的类型

1. 上面我们已经考虑的 `object、array` 都属于可以继续遍历的类型，因为它们内存都还可以存储其他数据类型的数据，另外还有 `Map，Set` 等都是可以继续遍历的类型，这里我们只考虑这四种，如果你有兴趣可以继续探索其他类型。

2. `Map，Set` 不能像数组对象一样增加属性，也不能使用 `for in` 遍历，所以：克隆 `Map，Set`：

```js

// 获取数据类型方法
const getType = (target) => {
  return Object.prototype.toString().call(target)
}

// set
if (getType(target) === '[object Set]') {
  target.forEach(item => cloneTarget.add(cloneDeep(item, map)))
}

// map，key可以为对象
if (getType(target) === '[object Map]') {
  target.forEach((item, key) => cloneTarget.set(cloneDeep(key, map), cloneDeep(item, map)))
}

```

##### 3. 不可继续遍历的类型

对于 `Bool、Number、String、Date、Error`、`Symbol`、正则、函数这些不可以继续遍历。

###### 1. `Bool、Number、String、Date、Error`包装器对象

这里指的是下面这种：

    ```js

    console.log(typeof new Number(1)) // object

    ```

这几种类型我们都可以直接用构造函数和原始数据创建一个新对象。

```js
// Bool、Number、String、Date、Error对象
let otherObj = [
  '[object Boolean]',
  '[object Number]',
  '[object String]',
  '[object Date]',
  '[object Error]'
]

if (otherObj.includes(getType(target))) {
  return new cloneTarget(target)
}

```

###### 2. 克隆Symbol包装器对象

1. 关于`Symbol`，看[Symbol](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol)

2. `Symbol.prototype.valueOf()`，返回当前 `symbol` 对象所包含的 `symbol` 原始值。

3. [聊一聊valueOf和toString](https://juejin.cn/post/6844903967097356302)

4. `Object` 构造函数将给定的值包装为一个新对象。

    -   如果给定的值是 null 或 undefined, 它会创建并返回一个空对象。
    -   否则，它将返回一个和给定的值相对应的类型的对象。
    -   如果给定值是一个已经存在的对象，则会返回这个已经存在的值（相同地址）。

在非构造函数上下文中调用时， `Object` 和 `new Object()` 表现一致。

###### 3. 克隆正则

###### 4. 克隆函数

1. 区分箭头函数和普通函数：通过 `prototype` ，箭头函数是没有 `prototype` 的。

2. 克隆箭头函数：直接使用 `eval` 和函数字符串来重新生成一个箭头函数，注意这种方法是不适用于普通函数的。

3. 克隆普通函数：分别使用正则取出函数体和函数参数，然后使用 `new Function ([arg1[, arg2[, ...argN]],] functionBody)` 构造函数重新构造一个新的函数。

4. 代码：

    ```js
    
    // 克隆函数
    if ('[object Function]') {
      return cloneFunction(target)
    }

    // 克隆函数方法
    function cloneFunction(func) {
      const bodyReg = /(?<={)(.|\n)+(?=})/m
      const paramReg = /(?<=\().+(?=\)\s+{)/
      const funcString = func.toString()

      if (func.prototype) {
          console.log('普通函数')
          const param = paramReg.exec(funcString)
          const body = bodyReg.exec(funcString)
          if (body) {
            console.log('匹配到函数体：', body[0])
            if (param) {
              const paramArr = param[0].split(',')
              console.log('匹配到参数：', paramArr)
              return new Function(...paramArr, body[0])
            } else {
              return new Function(body[0]);
            }
          } else {
            return null;
          }
      } else {
        return eval(funcString);
      }
    }

    ```

```js

const cloneDeep = (target, map = new WeakMap()) => {
  // Map 强引用，需要手动清除属性才能释放内存。
  // WeakMap 弱引用，随时可能被垃圾回收，使内存及时释放，是解决循环引用的不二之选。

  // null 和 undefined
  if (target === null || target === undefined) return target
  // 基本类型
  if (typeof target !== 'object' && typeof target !== 'function') return target
  // Date和RegExp对象
  if (target instanceof Date) return new Date(target)
  if (target instanceof RegExp) return new RegExp(target)
  // 函数和箭头函数
  if (typeof target === 'function') return handleFunction(target)

  // 对象
  // 循环引用
  if (map.get(target)) return map.get(target)
  let cloneObj = new target.constructor()
  map.set(target, cloneObj)

  // set
  if (getType(target) === '[object Set]') {
    target.forEach(item => cloneObj.add(cloneDeep(item, map)))
  }
  // map，key可以为对象
  if (getType(target) === '[object Map]') {
    target.forEach((item, key) => cloneObj.set(cloneDeep(key, map), cloneDeep(item, map)))
  }

  // 其他对象
  // Set和Map不能使用for in遍历
  for (let key in target) {
    if (target.hasOwnProperty(key)) {
      cloneObj[key] = cloneDeep(target[key], map)
    }
  }
}

// 拷贝函数的方法
const handleFunction = (target) => {
  // 箭头函数
  if (!target.prototype) return target

  const bodyReg = /(?<={)(.|\n)+(?=})/m
  const paramReg = /(?<=\().+(?=\)\s+{)/

  const funcString = target.toString()
  // 分别匹配 函数参数 和 函数体
  const param = paramReg.exec(funcString)
  const body = bodyReg.exec(funcString)
  if (!body) return null
  if (param) {
    const paramArr = param[0].split(',')
    return new Function(...paramArr, body[0])
  } else {
    return new Function(body[0])
  }
}


// todo 这个方法可以复制函数，待验证
if (target instanceof Function) {
  return function () {
    return target.apply(this, [...arguments]);
  };
}

const getType = (target) => {
  return Object.prototype.toString().call(target)
}

```

#### todo 性能优化

可以用 `while` 替换 `for in ` 遍历。

##### JS数组遍历的几种方式性能对比

1. 普通for循环

    ```js

    for(j = 0; j < arr.length; j++) {} 

    ```

    最简单的一种，也是使用频率最高的一种，虽然性能不弱，但仍有优化空间。

2. 优化版for循环

使用临时变量，将长度缓存起来，避免重复获取数组长度，当数组较大时优化效果才会比较明显。

    ```js

    for(j = 0, n = arr.length; j < n; j++) {} 

    ```

    所有循环遍历方法中性能最高的一种。

3. `foreach` 循环

    ```js

    arr.forEach(() => {})

    ```

    性能比普通for循环弱。

4. `foreach` 变种

    ```js

    Array.prototype.forEach.call(args, () => {})

    ```

    性能要比普通 `foreach` 弱

5. `for in` 循环

    ```js

    for(j in arr) {}

    ```

    效率是最低

6. `map` 遍历

    ```js

    arr.map(() => {})

    ```

    效率还比不上 `foreach`

7. `for of` 遍历

    ```js

    for(let j of arr) {}

    ```

    性能要好于 `for in`，但仍然比不上普通 `for` 循环

8. `while` 循环性能优于普通 `for` 循环

## 参考文献

1. [「中高级前端面试」手写代码合集](https://juejin.cn/post/6902060047388377095)
2. [如何写出一个惊艳面试官的深拷贝?](https://segmentfault.com/a/1190000020255831)
3. [JS几种数组遍历方式以及性能分析对比](https://dailc.github.io/2016/11/25/baseKnowlenge_javascript_jsarrayGoThrough)