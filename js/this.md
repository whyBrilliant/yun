# this

## this的四种绑定规则

### 一、默认绑定

> 规则：在非严格模式下，默认绑定的this指向全局对象，严格模式下this指向undefined

```
function foo() {
  console.log(this.a); // 2 this指向全局对象
}
var a = 2;
foo();

function foo2() {
  "use strict"; // 严格模式this绑定到undefined
  console.log(this.a); 
}
foo2(); // TypeError:a undefined
```

### 二、隐式绑定

> 规则：函数在调用位置，是否有上下文对象，如果有，那么this就会隐式绑定到这个对象上。

```
function foo() {
  console.log(this.a);
}
var a = "Oops, global";
let obj2 = {
  a: 2,
  foo: foo
};
let obj1 = {
  a: 22,
  obj2: obj2
};
obj2.foo(); // 2 this指向调用函数的对象
obj1.obj2.foo(); // 2 this指向最后一层调用函数的对象
```

> 隐式绑定丢失: 函数调用时，并没有上下文对象，只是对函数的引用，所以会导致隐式绑定丢失。

```
// 1:
function foo() {
  console.log(this.a);
}
var a = "Oops, global";
let obj2 = {
  a: 2,
  foo: foo
};
let bar = obj2.foo; // bar只是一个函数别名 是obj2.foo的一个引用
bar(); // "Oops, global" - 指向全局

// 2:
test(obj2.foo); // 传入函数的引用，调用时也是没有上下文对象。
```

### 三、显示绑定

>规则：在某个对象上强制调用函数，从而将this绑定在这个对象上。我们可以通过apply、call、bind将函数中的this绑定到指定对象上。

```
function foo() {
    console.log(this.a);
}
let obj = {
    a: 2
};
foo.call(obj); // 2
```

如果你传入了一个原始值(字符串,布尔类型，数字类型)，来当做this的绑定对象，这个原始值转换成它的对象形式。

如果你把null或者undefined作为this的绑定对象传入call/apply/bind，这些值会在调用时被忽略，实际应用的是默认绑定规则。

### 四、new绑定

在js中，实际上并不存在所谓的'构造函数'，只有对于函数的'构造调用'。

new的时候会做哪些事情：

1. 创建一个全新的对象。
2. 这个新对象会被执行 [[Prototype]] 连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

> 规则：使用构造调用的时候，this会自动绑定在new期间创建的对象上。

```
function foo(a) {
  this.a = a; 
}
let bar = new foo(2); // this绑定到bar上
console.log(bar.a); // 2
```

### 五、this四种绑定规则的优先级

显式绑定和new绑定无法直接比较(会报错),默认绑定是不应用其他规则之后的兜底绑定所以优先级最低

```
显式绑定 > 隐式绑定 > 默认绑定
new绑定 > 隐式绑定 > 默认绑定
```

示例：

```
obj.foo.call(obj2); // this指向obj2 显式绑定比隐式绑定优先级高。
new obj.foo(); // this指向new新创建的对象 new绑定比隐式绑定优先级高。
```

注意：箭头函数的this指向不会使用上述的四条规则
1. 箭头函数中的this继承于它外面第一个不是箭头函数的函数的this指向。
2. 箭头函数的 this 一旦绑定了上下文，就不会被任何代码改变。

参考： 《你不知道的JavaScript》

