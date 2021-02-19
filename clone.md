#### 浅拷贝
-   只拷贝一层对象
-   如果属性是基本类型，拷贝的就是基本类型的值，如果属性是引用类型，拷贝的就是内存地址
-   一般的数组和对象的赋值和方法都是浅拷贝，例如a = b，
Object.assign(a, b),array.concat()，[...a]

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
##### 判断对象属性是否存在
1. hasOwnProperty() 只能检测自有属性
2. in 可以检测自有属性和继承属性
3. 使用!==检测，使用！需要注意对象的属性值不能设置为undefined
   注意必须是！，而不是！= 因为！=不区分undefined和null

####深拷贝
-   深拷贝是将一个对象从内存中完整的拷贝一份出来,从堆内存中开辟一个新的区域存放新对象。
完全复制出一个对象，连地址也拷贝，两个不会相互影响。嵌套对象也完全拷贝。
-   乞丐版：
```js
JSON.parse(JSON.stringify(obj))
```
存在两个问题：
1. 无法解决循环引用 
```js
// TODO: 循环引用
```
2. 无法拷贝函数、RegExp、Date、Set、Map等特殊的对象。
-   工具库：lodash 的 cloneDeep 方法
-   手撕：
```js
const cloneDeep = () => {}
```
