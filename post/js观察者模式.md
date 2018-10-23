> 一年前实习时期的代码，今天拿出来总结梳理一下。
PS：以下代码基于es6实现，感兴趣的同学也可以用es5再重写一遍。
# 1. 观察者类
观察者模式有很多叫法，也叫发布订阅模式、监听者模式。
```
// 伪观察者
class Observer {
  constructor() {
    this._events = {}
  }
  on(type, fn) {
    this._events[type] = fn;
    return this;
  }
  trigger(type, ...args) {
    const fn  = this._events[type];
    if (fn) {
      fn.apply(this, args)
    }
    return this;
  }
  off(type) {
    delete this._events[type];
    this._events[type] = null;
    return this;
  }
}
```
以上所写是一个伪观察者的类（为什么是一个伪观察者呢，因为同名事件会相互覆盖，是观察者模式的一种简化实现方式），这个类有三个方法（on，trigger，off）分别功能是绑定事件，触发事件，和解绑事件的三个方法。每一个函数内都把this对象return是为了方便该对象的链式调用（可参考Jquery实现）。
接下来我们来写一个Btn类。

# 2. Btn类
```
// Button 类
class Btn extends Observer {
  constructor() {
    super()
    this.bindEvents();
    this.handleEvents();
  }
  handleEvents() {
    this.trigger('你追我如果你追上我', 'wo');
    return this;
  }
  bindEvents() {
    console.log('你追我如果你追上我');
    this.on('你追我如果你追上我', (who) => {
      console.log(who + '就让你嘿嘿嘿');
    })
    return this;
  }
}
```
非常简单的一个基于观察者模式的btn组件。
1. 在构造函数内我们先去调用super函数去继承Observer里面的方法和属性。
2. 调用bindEvents函数去绑定一个事件（当然最好是英文字符串，这里纯粹为了搞笑：P），这个时候'你追我如果你追上我'这个事件已经在_events对象内注册，只要去触发'你追我如果你追上我'这个事件就可以执行注册的回调函数。
3. 调用handleEvents函数，这也是一般的用户操作和其他事件的输入，我们可以通过trigger函数将我们需要触发的事件和参数一起传进去，然后Observer监听到事件之后就去执行当前事件下注册的回调函数。

# 3. 新建Btn对象
```
const button = new Btn();
button.trigger('你追我如果你追上我', 'ta');
```
```
$ node index.js
你追我如果你追上我
wo就让你嘿嘿嘿
ta就让你嘿嘿嘿
```
通过new的方式新建一个对象，该对象实现了Btn类，可以在Btn内触发trigger，也可以通过对象直接调用原型链上的trigger函数来触发当前事件。

# 4. 尾声
观察者模式可以说是在前端领域最最重要的一种设计模式，包括Redux，Vuex等数据管理模块，以及更基础的浏览器内部实现的事件监听，都是观察者模式的演变。