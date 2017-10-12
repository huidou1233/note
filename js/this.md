1 this是在调用时被绑定的，它指向什么取决于函数在哪里被调用。

2 绑定规则

  默认绑定

​	

```javascript
function foo(){
	console.log(this.a)
}
var a = 2;
foo();//2
```

调用foo()时，应用了this的默认绑定，因此this指向全局对象。

如果使用严格模式(strict mode)，那么全局对象将无法使用默认绑定，此时this会绑定到undefined



```javascript
 function foo(){
	"use strict"
     console.log(this.a)
 }
 var a = 2;
 foo();//Uncaught TypeError: Cannot read property 'a' of undefined
```
  隐式绑定



```javascript
  function foo(){
  	 console.log(this.a)
  }
  var obj = {
       a: 2,
       foo: foo
  }
  obj.foo();
```
当函数引用有上下文对象时，隐式绑定规则会把函数调用中的this绑定到这个上下文对象。调用foo()时，this被绑定到obj， 因此this.a和obj.a是一样的。

​	隐式丢失

​	常见的this绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定，从而把this绑定到全局对象或者underfined上，取决于是否是严格模式。如：

```javascript
   function foo() { 
        console.log( this.a );
    }
    function doFoo(fn) {
    // fn其实引用的是foo 
        fn(); // <-- 调用位置!
    }
    var obj = { 
        a: 2,
        foo: foo 
    };
    var a = "oops, global"; // a是全局对象的属性 
    doFoo( obj.foo );//"oops, global"
```
​	参数传递其实就是一种隐式赋值，因此我们传入函数式也会被隐式赋值，在doFoo中，obj.foo其实是一个不带任何修饰的函数调用，因此应用了默认绑定。

  显示绑定

​	使用call(…)和apply(...)进行显示绑定，如：

```javascript
	function foo() { 
        console.log( this.a );
    }
    var obj = { 
        a:2
    };
    foo.call( obj ); // 2
```
​	通过硬绑定解决绑定丢失的问题，如： 

```javascript
	function foo() { 
        console.log( this.a );
    }
    var obj = { 
        a:2
    };
    var bar = function() { 
        foo.call( obj );
    };
```
​	在函数bar()内部手动调用foo.call(obj),强制把foo的this绑定到了obj。无论之后如何调用bar，它总会手动在obj上调用foo。这种绑定是一种显式的强制绑定，因此被称为硬绑定。

  new绑定

使用new来调用函数，或者说发送构造调用时，会自动执行下面的操作：

​	1 创建(或者说构造)一个全新的对象

​	2 这个新对象会被执行[[原型]]连接

​	3 这个新对象会绑定到函数调用的this

​	4 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

3 优先级

​	1 如果函数是通过new绑定，this绑定的是新创建的对象

​		var bar = new foo()

​	2 如果函数式通过call、apply(显示绑定)或者硬绑定调用，this绑定的是指定的对象。

​		var bar = foo. call(obj2)

​	3 函数在某个上下文中调用(隐式绑定)，this绑定的是那个上下文对象。

​		var bar = obj1.foo()

​	4 如果都不是，使用默认绑定。如果在严格模式下，this绑定为undefined，否则绑定都全局对象

​		var bar = foo();

4 绑定例外

​	1 被忽略的this

​		如果将null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用时会被忽略，实际应用的是默认绑定规则：

```javascript
function foo() { 
     console.log( this.a );
}
var a = 2;
foo.call( null ); // 2
```
​	2 间接引用

​		有可能创建一个函数的"间接引用"， 在这种情况下，调用这个函数会应用默认绑定规则

```javascript
function foo() { 
	console.log( this.a );
}
var a = 2;
var o = { a : 3, foo: foo};
var p = { a : 4};
o.foo();// 3
(p.foo = o.foo)()// 2
```
​		赋值表达式 p.foo = o.foo 的返回值是目标函数的引用，因此调用位置是 foo() 而不是p.foo() 或者 o.foo()。根据我们之前说过的，这里会应用默认绑定。				

具体内容见《你不知道的JavaScript（上卷）》