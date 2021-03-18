实现思路：利用谁调用函数，函数的 this 就指向谁这一特点来实现。

```js
Function.prototype.myCall = function () {
  if (typeof this !== 'function') throw 'caller must be a function'

  let context = arguments[0] || window

  // 防止覆盖掉原有属性
  const fun = Symbol()
  context.fun = this

  // 参数
  let args = [] 

  Reflect.deleteProperty(context, 'fun') // 删除 _fn 属性
} 
```