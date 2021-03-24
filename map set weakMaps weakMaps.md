Map提供了与Object.values()和Object.entries() 等效的方法（只是它们返回Iterators），以便为Map实例提取属性值或键值对：


Map.prototype.values() 等价于Object.values()


Map.prototype.entries() 等价于Object.entries()


map是普通对象的改进版本，可以获取 map 的大小(对于普通对象，必须手动获取)，并使用任意对象类型作为键(普通对象使用字符串基元类型作为键)。
让我们看看返回.values（）和.entries（）的map的方法：
// ...
[...greetingsMap.values()];
// => ['Good morning', 'Good day', 'Good evening']
[...greetingsMap.entries()];
// => [ ['morning', 'Good morning'], ['midday', 'Good day'], 
//      ['evening', 'Good evening'] ]
注意，greetingsMap.values()和greetingsMap.entries()返回迭代器对象。若要将结果放入数组，扩展运算符…是必要的。


 [JS中轻松遍历对象属性的几种方式](https://juejin.cn/post/6844903906946842632)
 [js遍历集合（Array,Map,Set）](https://blog.csdn.net/wzj0808/article/details/51392060)