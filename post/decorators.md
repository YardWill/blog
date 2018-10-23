

## 元编程
在学习decorators之前，我们先来理解元编程的概念。Decorators是建立在元编程概念之上的。

1. 元编程是什么？
对于元编程的简单解释就是：代码检视自己，代码修改自己，或者代码修改默认的语言行为而使其他代码受影响
2. 借助哪些API来实现元编程？
在JS内我们现在有Proxy和Reflect来实现元编程。

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Meta_programming
## Proxy
Proxy是对一个对象进行代理，可以使在获取或者修改对象属性的时候，自定义的添加一些其他操作。

## Reflect
Reflect是一个工具集，这些工具一般与它们的Object.*对等物的行为相同。但一个区别是，Object.*对等物在它们的第一个参数值（目标对象）还不是对象的情况下，试图将它强制转换为一个对象。Reflect.*方法在同样的情况下仅简单地抛出一个错误。



## 生成器
1. 生成迭代器来控制生成器执行代码
2. yield来实现暂停
3. 使用return和throw提前结束和提前终止。
4. 使用regenerator重写generator，使用闭包的状态机

## 迭代器
1. API: next return throw  result {done, value}
2. ES6 很多重要部分的关键点。
3. 利用array[Symbol.iterator] 可以将数组转换成迭代器。

## Symbol
1. 通用Symbol 包括Symbol.iterator等 内置属性，可将XX转成迭代器。
2. Symbol仅仅只是一个标志，只是凸显他的唯一性，所有需要使用Symbol的地方都在原型上就冻好了。