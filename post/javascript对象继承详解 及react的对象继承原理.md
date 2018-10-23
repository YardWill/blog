>总结《javascript高级程序设计》第六章有关于对象继承的内容。看这篇文章之前看[js对象创建详解](http://www.jianshu.com/p/001b0b5148e0)。
对于OO语言的继承有两种形式，接口继承和实现继承，js中只支持实现继承。

对象继承的六种方法
===
1.原型链

```
function SuperType() {
    this.property = true;
}

SuperType.prototype.getSuperValue = function () {
    return this.property;
};

function SubType() {
    this.subproperty = false;
}

//inherit from SuperType
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function () {
    return this.subproperty;
};

var instance = new SubType();
alert(instance.getSuperValue());   //true

alert(instance instanceof Object);      //true
alert(instance instanceof SuperType);   //true
alert(instance instanceof SubType);     //true

alert(Object.prototype.isPrototypeOf(instance));    //true
alert(SuperType.prototype.isPrototypeOf(instance)); //true
alert(SubType.prototype.isPrototypeOf(instance));   //true
```
原型链是js实现继承的主要方法。其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。可以解决一般情况下的继承，但依旧存在问题：**正如对象创建一样，这种继承的引用类型值会被所有实例共享**。
**实际应用中很少单独使用原型链。**
2.借用构造函数（又叫伪造函数或经典继承）

```
function SuperType() {
    this.colors = ["red", "blue", "green"];
}

function SubType() {
    //继承SuperType
    SuperType.call(this);
}

//inherit from SuperType
SubType.prototype = new SuperType();

var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors);    //"red,blue,green,black"

var instance2 = new SubType();
alert(instance2.colors);    //"red,blue,green"
```
消除了原型链方法共享引用类型值的问题，**但也无法避免构造函数模式存在的问题---方法都在构造函数内定义，导致无法对函数进行复用**。

3.组合继承（又叫伪经典继承）

```
function SuperType(name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

SuperType.prototype.sayName = function () {
    alert(this.name);
};

function SubType(name, age) {
    SuperType.call(this, name);

    this.age = age;
}

//inherit from SuperType
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function(){
    alert(this.age);
}

var instance1 = new SubType('Nicholas', 29);
instance1.colors.push('black');
alert(instance1.colors);           //"red,blue,green,black"
instance1.sayName();               //'Nicholas'
instance1.sayAge();                //29

var instance2 = new SubType('Greg',27);
alert(instance2.colors);           //"red,blue,green"
instance2.sayName();               //'Greg'
instance2.sayAge();                //27
```
解决函数无法共用的问题。**属性用构造函数，函数用原型继承。**
**成为js中最常用的继承模式。**
4.原型式继承
```
function object(o) {
    function F() { }
    F.prototype = o;
    return new F();
}

var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = object(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

alert(person.friends);   //"Shelby,Court,Van,Rob,Barbie"
```
ES5中新增Object.create()替代该方法。

5.寄生式继承
```
function createAnother(o) {
    var clone = object(o);
    clone.sayHi = function() {
        alert('hi');
    }
    return clone;
}

var person = {
    name: 'Nicholas',
    friends: ['Shelby', 'Court', 'Van']
};

var anotherPerson = createAnother(person);
anotherPerson.sayHi();//'Hi'
```
与寄生构造函数和工厂模式相似。
**在主要考虑对象而不是自定义类型和构造函数时，寄生式继承是一种有用的模式。**

6.寄生组合式继承
```
function inheritPrototype(subType, superType){
    var prototype = Object(superType.prototype);
    prototype.constructor = subType;
    subType.property = prototype;
}

function SuperType(name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

SuperType.prototype.sayName = function () {
    alert(this.name);
};

function SubType(name, age) {
    SuperType.call(this, name);

    this.age = age;
}

inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function () {
    alert(this.age);
}
```
**是引用类型最理想的继承范式**

那么在让我们看看我们熟知的react是用什么方式来实现继承的。
实现继承的实例：
```
var CommentBox = React.createClass({
  render: function() {
    return (
      <div className="commentBox">
        Hello, world! I am a CommentBox.
      </div>
    );
  }
});
```
我们能观察到传入的是一个对象。
我们继续寻找React.createClass这个方法：（代码片段）
```
createClass: function (spec) {
  var Constructor = function (props, context, updater) {
    // This constructor gets overridden by mocks. The argument is used
    // by mocks to assert on what gets mounted.

    if (process.env.NODE_ENV !== 'production') {
      process.env.NODE_ENV !== 'production' ? warning(this instanceof Constructor, 'Something is calling a React component directly. Use a factory or ' + 'JSX instead. See: https://fb.me/react-legacyfactory') : void 0;
    }

    // Wire up auto-binding
    if (this.__reactAutoBindPairs.length) {
      bindAutoBindMethods(this);
    }

    this.props = props;
    this.context = context;
    this.refs = emptyObject;
    this.updater = updater || ReactNoopUpdateQueue;

    this.state = null;

    // ReactClasses doesn't have constructors. Instead, they use the
    // getInitialState and componentWillMount methods for initialization.

    var initialState = this.getInitialState ? this.getInitialState() : null;
    if (process.env.NODE_ENV !== 'production') {
      // We allow auto-mocks to proceed as if they're returning null.
      if (initialState === undefined && this.getInitialState._isMockFunction) {
        // This is probably bad practice. Consider warning here and
        // deprecating this convenience.
        initialState = null;
      }
    }
    !(typeof initialState === 'object' && !Array.isArray(initialState)) ? process.env.NODE_ENV !== 'production' ? invariant(false, '%s.getInitialState(): must return an object or null', Constructor.displayName || 'ReactCompositeComponent') : _prodInvariant('82', Constructor.displayName || 'ReactCompositeComponent') : void 0;

    this.state = initialState;
  };
  Constructor.prototype = new ReactClassComponent();
  Constructor.prototype.constructor = Constructor;
  Constructor.prototype.__reactAutoBindPairs = [];

  injectedMixins.forEach(mixSpecIntoComponent.bind(null, Constructor));

  mixSpecIntoComponent(Constructor, spec);

  // Initialize the defaultProps property after all mixins have been merged.
  if (Constructor.getDefaultProps) {
    Constructor.defaultProps = Constructor.getDefaultProps();
  }

  if (process.env.NODE_ENV !== 'production') {
    // This is a tag to indicate that the use of these method names is ok,
    // since it's used with createClass. If it's not, then it's likely a
    // mistake so we'll warn you to use the static property, property
    // initializer or constructor respectively.
    if (Constructor.getDefaultProps) {
      Constructor.getDefaultProps.isReactClassApproved = {};
    }
    if (Constructor.prototype.getInitialState) {
      Constructor.prototype.getInitialState.isReactClassApproved = {};
    }
  }

  !Constructor.prototype.render ? process.env.NODE_ENV !== 'production' ? invariant(false, 'createClass(...): Class specification must implement a `render` method.') : _prodInvariant('83') : void 0;

  if (process.env.NODE_ENV !== 'production') {
    process.env.NODE_ENV !== 'production' ? warning(!Constructor.prototype.componentShouldUpdate, '%s has a method called ' + 'componentShouldUpdate(). Did you mean shouldComponentUpdate()? ' + 'The name is phrased as a question because the function is ' + 'expected to return a value.', spec.displayName || 'A component') : void 0;
    process.env.NODE_ENV !== 'production' ? warning(!Constructor.prototype.componentWillRecieveProps, '%s has a method called ' + 'componentWillRecieveProps(). Did you mean componentWillReceiveProps()?', spec.displayName || 'A component') : void 0;
  }

  // Reduce time spent doing lookups by setting these on the prototype.
  for (var methodName in ReactClassInterface) {
    if (!Constructor.prototype[methodName]) {
      Constructor.prototype[methodName] = null;
    }
  }

  return Constructor;
}
```
可以看出是使用寄生构造函数模式创建的对象，而继承方式是寄生组合式继承。


>总结：一般情况下，基本类型使用构造函数继承，引用类型（包括函数）使用原型继承，通常情况下组合继承。此处只是做一个对于对象继承的归类与总结，具体的内容与解释都在《js高级程序设计》第六章上，如果想要深入了解，强烈推荐去阅读这本书。