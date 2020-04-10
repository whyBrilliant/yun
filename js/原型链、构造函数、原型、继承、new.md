# 原型链、构造函数、原型、继承、new

## 原型链

> 原型链：每个对象拥有一个原型对象，通过__proto__指针指向上一个原型，并从中继承方法和属性，同时原型对象也可能拥有原型，这样一层一层，最终指向null,这就是原型链。

```
function Parent(age) {
    this.age = age;
}

var p = new Parent(50);
p.constructor === Parent; // true

```

![](https://github.com/whyBrilliant/yun/blob/master/img/prototype-2.jpg)

由图可以看到实例对象 p 本身没有 constructor 属性，是通过原型链向上查找 \_\_proto__'，最终查找到 constructor 属性，该属性指向 Parent。

```
function Parent(age) {
    this.age = age;
}
var p = new Parent(50);

p;	// Parent {age: 50}
p.__proto__ === Parent.prototype; // true
p.__proto__.__proto__ === Object.prototype; // true
p.__proto__.__proto__.__proto__ === null; // true
```
![]('https://github.com/whyBrilliant/yun/blob/master/img/prototype-4.jpg')


## 构造函数

>构造函数本身就是一个函数，与普通函数没有任何区别，不过为了规范一般将其首字母大写。构造函数和普通函数的区别在于，使用 new 生成实例的函数就是构造函数，直接调用的就是普通函数。

>constructor 返回创建实例对象时构造函数的引用。此属性的值是对函数本身的引用，而不是一个包含函数名称的字符串。

### Symbol 是构造函数吗

Symbol 是基本数据类型，但作为构造函数来说它并不完整，因为它不支持语法 new Symbol()，Chrome 认为其不是构造函数，如果要生成实例直接使用 Symbol() 即可。（来自 MDN）虽然是基本数据类型，但 Symbol(123) 实例可以获取 constructor 属性值.

```
new Symbol(123); // Symbol is not a constructor 

Symbol(123); // Symbol(123)

var sym = Symbol(123); 
console.log( sym );
// Symbol(123)

console.log( sym.constructor );
// ƒ Symbol() { [native code] }
```

### constructor 值只读吗 ?
这个得分情况，对于引用类型来说 constructor 属性值是可以修改的，但是对于基本类型来说是只读的。

引用类型情况其值可修改这个很好理解，比如原型链继承方案中，就需要对 constructor重新赋值进行修正。

```
function Foo() {
    this.value = 42;
}
Foo.prototype = {
    method: function() {}
};

function Bar() {}

// 设置 Bar 的 prototype 属性为 Foo 的实例对象
Bar.prototype = new Foo();
Bar.prototype.foo = 'Hello World';

Bar.prototype.constructor === Object;
// true

// 修正 Bar.prototype.constructor 为 Bar 本身
Bar.prototype.constructor = Bar;

var test = new Bar() // 创建 Bar 的一个新实例
console.log(test);

```

对于基本类型来说是只读的，比如 Number、String、Boolean、Symbol，当然 null 和 undefined 是没有 constructor 属性的。

```
function Type() { };
var	types = [1, "muyiy", true, Symbol(123)];

for(var i = 0; i < types.length; i++) {
	types[i].constructor = Type;
	types[i] = [ types[i].constructor, types[i] instanceof Type, types[i].toString() ];
};

console.log( types.join("\n") );
// function Number() { [native code] }, false, 1
// function String() { [native code] }, false, muyiy
// function Boolean() { [native code] }, false, true
// function Symbol() { [native code] }, false, Symbol(123)

```

## 原型

> prototype: 每个对象拥有一个原型对象，对象以其原型为模板，从原型继承方法和属性，这些属性和方法定义在对象的构造器函数的 prototype 属性上，而非对象实例本身。

![]('https://github.com/whyBrilliant/yun/blob/master/img/prototype.jpg')

从上面这张图可以发现，Parent 对象有一个原型对象 Parent.prototype，其上有两个属性，分别是 constructor 和 \_\_proto__，其中 \_\_proto__ 已被弃用。

构造函数 Parent 有一个指向原型的指针，原型 Parent.prototype 有一个指向构造函数的指针 Parent.prototype.constructor，如上图所示，其实就是一个循环引用。

![]('https://github.com/whyBrilliant/yun/blob/master/img/prototype-1.jpg')

> \_\_proto__ 是每个实例上都有的属性, 这是一个访问器属性（即 getter 函数和 setter 函数），通过它可以访问到对象的内部 [[Prototype]] (一个对象或 null )

![]('https://github.com/whyBrilliant/yun/blob/master/img/prototype-2.jpg')

这里用 p.\_\_proto__ 获取对象的原型，prototype 是构造函数的属性，p.\_\_proto__ 和 Parent.prototype 指向同一个对象。

```
function Parent() {}
var p = new Parent();
p.__proto__ === Parent.prototype // true

```

如果要读取或修改对象的 [[Prototype]] 属性，建议使用如下方案，但是此时设置对象的 [[Prototype]] 依旧是一个缓慢的操作，如果性能是一个问题，就要避免这种操作。

```
// 获取
Object.getPrototypeOf()
Reflect.getPrototypeOf()

// 修改
Object.setPrototypeOf()
Reflect.setPrototypeOf()
```

如果要创建一个新对象，同时继承另一个对象的 [[Prototype]] ，推荐使用 Object.create()。

```
function Parent() {
    age: 50
};
var p = new Parent();
var child = Object.create(p);
```
这里 child 是一个新的空对象，有一个指向对象 p 的指针 \_\_proto__。


#### 小结
- Symbol 作为构造函数来说并不完整，因为不支持语法 new Symbol()，但其原型上拥有 constructor 属性，即 Symbol.prototype.constructor。
- 引用类型 constructor 属性值是可以修改的，但是对于基本类型来说是只读的，当然 null 和 undefined 没有 constructor 属性。
- \_\_proto__ 是每个实例上都有的属性，prototype 是构造函数的属性，这两个并不一样，但  p.__proto__ 和 Parent.prototype 指向同一个对象。
- \_\_proto__ 属性在 ES6 时被标准化，但因为性能问题并不推荐使用，推荐使用 Object.getPrototypeOf()。
- 每个对象拥有一个原型对象，通过 \_\_proto__ 指针指向上一个原型 ，并从中继承方法和属性，同时原型对象也可能拥有原型，这样一层一层，最终指向 null，这就是原型链。

#### 参考
- 掘金 木易杨说 [【进阶5-1期】重新认识构造函数、原型和原型链](https://juejin.im/post/5c6a9c10f265da2db87b98f3)


## 继承的八种方式 ?

### 一. 原型链法（使用原型）

基本思想是利用原型让一个引用类型继承另一个引用类型的方法和实例。

```
function staff(){ 
  this.company = 'ABC';
}
staff.prototype.companyName = function(){
  return this.company; 
}
function employee(name,profession){
  this.employeeName = name;
  this.profession = profession;
}
// 继承 staff
employee.prototype = new staff();
// 将这个对象的 constructor 手动改成 employee，否则还会是 staff
employee.prototype.constructor = employee;
// 不使用对象字面量方式创建原型方法，会重写原型链
employee.prototype.showInfo = function(){
  return this.employeeName + "'s profession is " + this.profession;
}
let instance = new employee('Andy','front-end');

// 测试 
console.log(instance.companyName()); // ABC
console.log(instance.showInfo());    // "Andy's profession is front-end"
// 通过 hasOwnProperty() 方法来确定自身属性与其原型属性
console.log(instance.hasOwnProperty('employeeName'))     // true
console.log(instance.hasOwnProperty('company'))          // false
// 通过 isPrototypeOf() 方法来确定原型和实例的关系
console.log(employee.prototype.isPrototypeOf(instance)); // true
console.log(staff.prototype.isPrototypeOf(instance));    // true
console.log(Object.prototype.isPrototypeOf(instance));   // true

```

#### 存在的问题

>当原型中存在引用类型值时，实例可以修改其值。

```
function staff(){ 
  this.test = [1,2,3,4];
}
function employee(name,profession){
  this.employeeName = name;
  this.profession = profession;
}
employee.prototype = new staff();
let instanceOne = new employee();
let instanceTwo = new employee();
instanceOne.test.push(5);
console.log(instanceTwo.test); // [1, 2, 3, 4, 5]
```

#### 小结
1. 基于构造函数和原型链
2. 通过 hasOwnProperty() 方法来确定自身属性与其原型属性
3. 通过 isPrototypeOf() 方法来确定原型和实例的关系
4. 在实例中可以修改原型中引用类型的值

### 二. 仅继承父构造函数的原型对象

```
function staff(){ }
staff.prototype = { test: [1, 2, 3, 4] }
function employee(name,profession){
  this.employeeName = name;
  this.profession = profession;
}
employee.prototype = staff.prototype;
var instanceOne = new employee();
var instanceTwo = new employee();
instanceOne.test.push(5);
console.log(instanceTwo.test); // [1, 2, 3, 4, 5]
```

#### 优点

1. 构建继承关系时不需要新建对象实例
2. 由于公用一个原型对象，所以在访问对象的时候不需要遍历原型链，效率自然就高

#### 缺点

1. 和方法一相同，子对象的修改会影响父对象。

#### 小结

1. 基于构造函数，没有使用原型链
2. 子对象和父对象公用一个原型对象

### 三. 借用构造函数法

```
function staff(){ 
  this.test = [1,2,3];
}
staff.prototype.companyName = function(){
  return this.company; 
}
function employee(name,profession){
  staff.call(this);	
  this.employeeName = name;
  this.profession = profession;
}
// 不使用对象字面量方式创建原型方法，会重写原型链
employee.prototype.showInfo = function(){
  return this.employeeName + "'s profession is " + this.profession;
}
let instanceOne = new employee('Andy','front-end');
let instanceTwo = new employee('Mick','after-end');
instanceOne.test.push(4);
// 测试 
console.log(instanceTwo.test);    // [1,2,3]
// console.log(instanceOne.companyName()); // 报错
// 通过 hasOwnProperty() 方法来确定自身属性与其原型属性
console.log(instanceOne.hasOwnProperty('test'))          // true
// 通过 isPrototypeOf() 方法来确定原型和实例的关系
console.log(staff.prototype.isPrototypeOf(instanceOne));    // false
```

#### 小结

1. 只能继承父对象的实例属性和方法，不能继承父对象原型属性和方法
2. 无法实现函数复用，每个子对象都有父对象实例的副本，性能欠优

### 四. 组合继承

指的是将原型链技术和借用构造函数技术结合起来，二者皆取其长处的一种经典继承方式。

```
function staff(){ 
  this.company = "ABC";	
  this.test = [1,2,3];
}
staff.prototype.companyName = function(){
  return this.company; 
}
function employee(name,profession){
  // 继承属性
  staff.call(this);	
  this.employeeName = name;
  this.profession = profession;
}
// 继承方法
employee.prototype = new staff();
employee.prototype.constructor = employee;
employee.prototype.showInfo = function(){
  return this.employeeName + "'s profession is " + this.profession;
}

let instanceOne = new employee('Andy','front-end');
let instanceTwo = new employee('Mick','after-end');
instanceOne.test.push(4);
// 测试 
console.log(instanceTwo.test);    // [1,2,3]
console.log(instanceOne.companyName()); // ABC
// 通过 hasOwnProperty() 方法来确定自身属性与其原型属性
console.log(instanceOne.hasOwnProperty('test'))          // true
// 通过 isPrototypeOf() 方法来确定原型和实例的关系
console.log(staff.prototype.isPrototypeOf(instanceOne));    // true
```

#### 优点

1. 可以复用原型上定义的方法
2. 可以保证每个函数有自己的属性，可以解决原型中引用类型值被修改的问题

#### 缺点

1. staff 会被调用 2 次：第 1 次是employee.prototype = new staff();，第 2 次是调用 staff.call(this)。


### 五. 原型式继承 - Object.create()

利用一个临时性的构造函数（空对象）作为中介，将某个对象直接赋值给构造函数的原型。

```
function object(obj){
  function F(){}
  F.prototype = obj;
  return new F();
}
```
本质上 object() 对传入其中的对象执行了一次浅复制，将构造函数 F 的原型直接指向传入的对象。

```
var employee = {
  test: [1,2,3]
}

let instanceOne = object(employee);
let instanceTwo = object(employee);
// 测试 
instanceOne.test.push(4);
console.log(instanceTwo.test); // [1, 2, 3, 4]
```

#### 缺点
1. 原型中引用类型值会被修改
2. 无法传递参数


### 六. 寄生式继承

在原型式继承的基础上，通过封装继承过程的函数增强对象，返回对象

```
function createAnother(original){
  var clone = object(original); // 通过调用 object() 函数创建一个新对象
  clone.sayHi = function(){  // 以某种方式来增强对象
    alert("hi");
  };
  return clone; // 返回这个对象
}
```
createAnother 函数的主要作用是为构造函数新增属性和方法，以增强函数。

#### 缺点
1. 原型中引用类型值会被修改
2. 无法传递参数

### 七. 寄生组合式继承

该方法主要是解决组合继承调用两次超类构造函数的问题。

```
function inheritPrototype(sub, super){
  var prototype = Object.create(super.prototype); // 创建对象，父原型的副本
  prototype.constructor = sub;                    // 增强对象
  sub.prototype = prototype;                      // 指定对象，赋给子的原型
}

function staff(){ 
  this.company = "ABC";	
  this.test = [1,2,3];
}
staff.prototype.companyName = function(){
  return this.company; 
}
function employee(name,profession){
  staff.call(this, name);
  this.employeeName = name;
  this.profession = profession;
}

// 将父类原型指向子类
inheritPrototype(employee,staff)
let instanceOne = new employee("Andy", "A");
let instanceTwo = new employee("Rose", "B");
instanceOne.test.push(4);
// 测试 
console.log(instanceTwo.test);            // [1,2,3]
console.log(instanceOne.companyName());   // ABC
// 通过 hasOwnProperty() 方法来确定自身属性与其原型属性
console.log(instanceOne.hasOwnProperty('test'))           // true
// 通过 isPrototypeOf() 方法来确定原型和实例的关系
console.log(staff.prototype.isPrototypeOf(instanceOne));  // true
```

### 八.Class 的继承

Class 可以通过 extends 关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多。

```
class staff { 
  constructor(){
    this.company = "ABC";	
    this.test = [1,2,3];
  }
  companyName(){
    return this.company; 
  }
}
class employee extends staff {
  constructor(name,profession){
    super();
    this.employeeName = name;
    this.profession = profession;
  }
}

// 将父类原型指向子类
let instanceOne = new employee("Andy", "A");
let instanceTwo = new employee("Rose", "B");
instanceOne.test.push(4);
// 测试 
console.log(instanceTwo.test);    // [1,2,3]
console.log(instanceOne.companyName()); // ABC
// 通过 Object.getPrototypeOf() 方法可以用来从子类上获取父类
console.log(Object.getPrototypeOf(employee) === staff)
// 通过 hasOwnProperty() 方法来确定自身属性与其原型属性
console.log(instanceOne.hasOwnProperty('test'))          // true
// 通过 isPrototypeOf() 方法来确定原型和实例的关系
console.log(staff.prototype.isPrototypeOf(instanceOne));    // true
```

> 1. 子类必须在 constructor 方法中调用 super 方法，否则新建实例时会报错。这是因为子类没有自己的this 对象，而是继承父类的 this 对象，然后对其进行加工。
> 2. 只有调用 super 之后，才可以使用 this 关键字，否则会报错。这是因为子类实例的构建，是基于对父类实例加工，只有 super 方法才能返回父类实例。

- super` 虽然代表了父类 `A` 的构造函数，但是返回的是子类 `B` 的实例，即` super` 内部的 `this ` 指的是 `B`，因此 `super()` 在这里相当于 A.prototype.constructor.call(this)


#### ES5 和 ES6 实现继承的区别
    ES5 的继承，实质是先创造子类的实例对象 this，然后再将父类的方法添加到 this 上面（Parent.apply(this)）。
    ES6 的继承机制完全不同，实质是先创造父类的实例对象 this （所以必须先调用 super() 方法），然后再用子类的构造函数修改 this。

extends 继承核心代码（寄生组合式继承）

```
function _inherits(subType, superType) {
  subType.prototype = Object.create(superType && superType.prototype, {
    constructor: {
      value: subType,
      enumerable: false,
      writable: true,
      configurable: true
    }
  });
  if (superType) {
    Object.setPrototypeOf 
    ? Object.setPrototypeOf(subType, superType) 
    : subType.__proto__ = superType;
  }
}
```
1. 子类的 \_\_proto__ 属性，表示构造函数的继承，总是指向父类。
2. 子类 prototype 属性的 \_\_proto__ 属性，表示方法的继承，总是指向父类的 prototype 属性。

#### 参考
- 掘金 小生方勤 [【前端词典】继承（二） - 回的八种写法·面试必问](https://juejin.im/post/5c40408bf265da61193c1606)
