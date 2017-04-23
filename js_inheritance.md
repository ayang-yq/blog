JavaScript 常用继承模式!
===================
本文探讨了JavaScript中实现继承的几种模式及其演化，主要参考《JavaScript高级程序设计》。

### 原型链模式（Prototype Chaining）
每个构造函数都有一个原型对象(prototype)，原型对象都包含一个指向构造函数的指针(constructor)，而实例都包含一个指向原型对象的内部指针\[\[Prototype\]\]。那么让原型对象等于另一个类型的实例，即可实现基于原型的继承。
```javascript
function SuperType() {
    this.property = "superType";
}

SuperType.prototype.getSuperValue = function () {
    return this.property;
}

function SubType() {
    this.subProperty = "subType";
}

//继承SuperType
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function () {
    return this.subProperty;
}

var instance = new SubType();
console.log(instance.getSuperValue()); //superType

//Instance -> SubType Prototype -> SuperType Prototype
```
测试原型和实例的关系
```javascript
console.log(instance instanceof Object);        //true
console.log(instance instanceof SuperType);     //true
console.log(instance instanceof SubType);       //true

console.log(Object.prototype.isPrototypeOf(instance));      //true
console.log(SuperType.prototype.isPrototypeOf(instance));   //true
console.log(SubType.prototype.isPrototypeOf(instance));     //true
```
> 原型链的缺点:
> 原型链最大的问题来自包含引用类型的原型。比如从父类型继承了一个数组，子类型的实例都将引用到同一个数组。

### 借用构造函数（Constructor Stealing）
```javascript
function SuperType() {
    this.colors = ["red", "blue", "green"];
}

function SubType() {
    //继承SuperType
    SuperType.call(this);
}

var instance1 = new SubType();
instance1.colors.push("black");
console.log(instance1.colors); //"red,blue,green,black"

var instance2 = new SubType();
console.log(instance2.colors); //"red,blue,green"

console.log(instance1 instanceof SubType);      //true
console.log(instance1 instanceof SuperType);    //false
```
>借用构造函数的问题：
>方法没有办法复用，在父类型中定义的方法对子类型也不可见。同时也不能用instanceof来检测实例之间的继承关系。

### 组合继承模式（Combination Inheritance）
```javascript
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function () {
    console.log(this.name);
}

function SubType(name, age) {
    SuperType.call(this, name);
    this.age = age;
}

SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function() {
    console.log(this.age);
}

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
console.log(instance1.colors);  //"red,blue,green,black"
instance1.sayName();    //Nicholas
instance1.sayAge();     //29

var instance2 = new SubType("Greg", 27);
console.log(instance2.colors);  //"red,blue,green"
instance2.sayName();    //Greg
instance2.sayAge();     //27
```
>组合继承模式的缺点：
>该模式解决了原型链继承和借用构造函数的不足，但是仔细观察我们可以发现父类型的构造函数被调用了两次，在调用子类型构造函数的时候必须重写从父类型继承的属性。

### 原型式继承（Prototypal Inheritance）
Douglas Crockford在2006年的一片文章介绍了这种继承实现的方法。本质上是object对传入的对象执行了一次浅复制，如果传入的对象包含引用类型，使用该函数创建的实例仍然会共享同一个引用对象。
```javascript
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}
```
ECMAScript5通过新增Object.create()方法规范了原型式继承，这个方法接受两个参数：一个用作新对象原型的对象和一个为新对象定义额外属性的对象。
```
Object.create(proto[, propertiesObject])
```

```javascript
var person = {
    name: "Nicolas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = Object.create(person, {
    name: {
        value: "Greg"
    }
});
console.log(anotherPerson.name);    //Greg
```
>原型式继承的缺点：
>父类型中定义的引用类型仍然会被子类型实例共享。

### 寄生组合式继承（Parasitic Combination Inheritance）
```javascript
///************************************
/// 1. 创建父类型原型的一个副本
/// 2. 为创建的副本添加constructor属性，从而弥补因重写原型而失去的默认的constructor属性
/// 3. 将新创建的对象赋值给子类型的原型
///************************************
function inheritPrototype(subType, superType) {
    var prototype = Object.create(superType.prototype);
    prototype.constructor = subType;
    subType.prototype = prototype;
}

function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function () {
    console.log(this.name);
}

function SubType(name, age) {
    //继承父类型的属性
    SuperType.call(this, name);
    this.age = age;
}

//继承父类型的方法
inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function () {
    console.log(this.age);
}

var instance1 = new SubType("Greg", 27);
instance1.colors.push("black");

var instance2 = new SubType("Douglas", 29);

console.log(instance1.sayName);  //Greg
console.log(instance1.sayAge);   //27

console.log(instance1.colors);      //"red,blue,green,black"
console.log(instance2.colors);      //"red,blue,green"

console.log(instance1 instanceof SubType);   //true
console.log(instance1 instanceof SuperType); //true
```
更常见的写法
```javascript
function SuperType(name) {
    this.name = name;
}

SuperType.prototype.sayName = function () {
    console.log(this.name);
}

function SubType(name, age) {
    SuperType.call(this, name);
    this.age = age;
}

SubType.prototype = Object.create(SuperType.prototype);
SubType.prototype.constructor = SubType;

var instance = new SubType("Greg", 27);
```
>寄生组合式继承的优点：
>只调用一次父类型的构造函数，子类型的原型指向的是父类型原型的一个副本而不是父类型的实例，因此避免了在子类型的原型上创建多余的属性，同时保持了原型链。

综上所诉，寄生组合式继承是比较完善的一种继承实现方式，在工作中也是比较常用的。


