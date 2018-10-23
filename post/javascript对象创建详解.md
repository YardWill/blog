>总结《javascript高级程序设计》第六章有关于对象创建的内容

对象创建的七种模式
====
1.工厂模式

```
function createPerson(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function () {
        alert(this.name);
    };
    return o;
}

var person1 = createPerson("Nicholas", 29, "Software Engineer");
var person2 = createPerson("Greg", 27, "Doctor");

person1.sayName();   //"Nicholas"
person2.sayName();   //"Greg"

```
通过函数创建一个新对象，然后返回这个新对象到想要赋值的变量内。
>1. 解决问题： 这种模式抽象了创建对象的过程，用来封装js中的“类”。
2. 引入问题：无法解决对象识别问题（即怎样知道对象的类型）。

2.构造函数模式

```
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function () {
        alert(this.name);
    };
}

var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```
通过new来调用构造函数来给对象赋值。
>1. 解决问题： 可以使用instanceof来识别对象的类型。
2. 引入问题：每一次调用构造函数都会重新创建sayName（引用类型）实例。无法共享sayName函数，导致内存上的浪费。

3.原型模式

```
function Person() {
}

Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function () {
    alert(this.name);
};

var person1 = new Person();
person1.sayName();   //"Nicholas"

var person2 = new Person();
person2.sayName();   //"Nicholas"

alert(person1.sayName == person2.sayName);  //true

alert(Person.prototype.isPrototypeOf(person1));  //true
alert(Person.prototype.isPrototypeOf(person2));  //true

//only works if Object.getPrototypeOf() is available
if (Object.getPrototypeOf) {
    alert(Object.getPrototypeOf(person1) == Person.prototype);  //true
    alert(Object.getPrototypeOf(person1).name);  //"Nicholas"
}
```

通过prototype（原型）属性去创建对象 。
>1. 解决问题： 共享sayName函数，很好的对象封装性。
2. 引入问题：原型的属性共用，如果属性为引用变量，将导致不同实例共用同一个属性，导致数据出现问题。（非引用变量将直接“覆盖”）

4.组合使用构造函数和原型模式
```
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ["Shelby", "Court"];
}

Person.prototype = {
    constructor: Person,
    sayName: function () {
        alert(this.name);
    }
};

var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");

person1.friends.push("Van");

alert(person1.friends);    //"Shelby,Court,Van"
alert(person2.friends);    //"Shelby,Court"
alert(person1.friends === person2.friends);  //false
alert(person1.sayName === person2.sayName);  //true
```

>1. 解决问题： 防止公用引用属性联动，属性使用构造函数模式，函数使用原型模式。
**是目前ES中使用最广泛、认同度最高的一种创建自定义类型的方法。**

5.动态原型模式
```
function Person(name, age, job) {

    //properties
    this.name = name;
    this.age = age;
    this.job = job;

    //methods
    if (typeof this.sayName != "function") {

        Person.prototype.sayName = function () {
            alert(this.name);
        };

    }
}

var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();
```
原理类似原型模式，当sayName为函数是使用原型去定义。

6.寄生构造函数模式

```
function Person(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function () {
        alert(this.name);
    };
    return o;
}

var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();  //"Nicholas"
```
类似工厂模式，将函数变成一个构造函数，通过new进行调用。
**在可以使用其他模式的时候，不要使用这种模式。**

7.稳妥构造函数模式

```
function Person(name, age, job) {
    var o = new Object();
    oname = name;
    oage = age;
    ojob = job;
    o.sayName = function () {
        alert(oname);
    };
    return o;
}

var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();  //"Nicholas"
```
类似寄生构造函数模式，**唯一不同的是无法通过对象直接调用内部属性，只能通过内置方法调用，提高系统安全性。**