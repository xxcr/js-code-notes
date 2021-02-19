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



#深拷贝
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

#### 简易版
```js
const cloneDeep = (target) => {
  // 定义一个变量
  let result;
  // 如果当前需要深拷贝的是一个对象的话
  if (typeof target === 'object') {
    // 如果是一个数组的话
    if (Array.isArray(target)) {
      result = []; // 将result赋值为一个数组，并且执行遍历
      for (let i in target) {
        result.push(deepClone(target[i]))
      }
     // 判断如果当前的值是null的话；直接赋值为null
    } else if(target===null) {
      result = null;
   // 判断如果当前的值是一个RegExp对象的话，直接赋值    
    } else if(target.constructor===RegExp){
      result = target;
    } else {
     // 否则是普通对象，直接for in循环，递归赋值对象的所有值
      result = {};
      for (let i in target) {
        result[i] = deepClone(target[i]);
      }
    }
   // 如果不是对象的话，就是基本数据类型，那么直接赋值
  } else {
    result = target;
  }
  return result;
}
```

#### 完整版
```js
const cloneDeep = (target, map = new WeakMap()) => {
  
}
```
