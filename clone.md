# 赋值
a = b
1. 基本数据类型：两个变量不影响
2. 引用数据类型：只复制地址，两个变量指向同一个地址，a和b同一个对象

[js数据类型及判断](https://github.com/xxcr/js-code-notes/blob/main/data-types.md)



# 浅拷贝
1. 与赋值不同，拷贝一层对象
2. 如果属性是基本类型，拷贝的就是基本类型的值，如果属性是引用类型，拷贝的就是内存地址
3. 一般的数组和对象的赋值和方法都是浅拷贝，例如：
	-   <font color="#dd0000">Object.assign(a, b)</font>：把所有可枚举属性从一个或多个对象复制到目标对象，返回目标对象。
	-   array.concat()、Array.prototype.slice()
	-   [...a]

```js
const shallowClone = (target) => {
  if (typeof target === 'object' && target !== null) { // 对象，数组，函数，但不为null
    const cloneTarget = Array.isArray(target) ? [] : {}
    for (item in target) { 
      if (target.hasOwnProperty(prop)) { // 是否是自身属性（非继承）
        cloneTarget[prop] = target[prop]
      }
    }
    return cloneTarget
  } else {
    return target // 基础类型直接返回
  }
}
```

### 判断对象属性是否存在
1. hasOwnProperty() 只能检测自有属性
2. in 可以检测自有属性和继承属性
3. 使用!==检测，使用！需要注意对象的属性值不能设置为undefined
   注意必须是！，而不是！= 因为！=不区分undefined和null


## 和赋值区别
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
console.log(b) // {name: 4, address: {city: 5, home: 3}}
console.log(c) // {name: 1, address: {city: 5, home: 3}}

```



# 深拷贝
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
	
		[循环引用](https://github.com/xxcr/js-code-notes/blob/main/循环引用.md)
	-   无法拷贝函数、RegExp、Date、Set、Map等特殊的对象。
	-   会忽略undefined、symbol。
	-   不能序列化函数

4. 工具库：lodash 的 cloneDeep 方法

#### 简易易记版
```js
const cloneDeep = (target) => {
  let result;
  // 如果当前需要深拷贝的是一个对象的话
  if (typeof target === 'object') {
    // 数组
    if (Array.isArray(target)) {
      result = [];
      for (let i in target) {
        result.push(deepClone(target[i]))
      }
     // 判断如果当前的值是null的话；直接赋值为null
    } else if(target===null) {
      result = null;
    } else {
     // 普通对象，直接for in循环，递归赋值对象的所有值
      result = {};
      for (let i in target) {
        result[i] = deepClone(target[i]);
      }
    }
   // 基本数据类型，直接赋值
  } else {
    result = target;
  }
  return result;
}
```

#### 完整版
参照  https://juejin.cn/post/6902060047388377095
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

const getType = (target) => {
  return Object.prototype.toString().call(target)
}
```
