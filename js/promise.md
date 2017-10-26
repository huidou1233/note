1  Promise API

 1 new Promise() 构造器

​	Promise()必须和new一起使用，并且提供一个函数回调。这个回调是同步的或立即调用的。这个函数必须接受两个函数回调，用以支持promise的决议。通常把这两个函数称为resolve(…)和reject(...)

​	var  p = new Promise( function(resolve, reject){

​		//resolve()用于决议/完成这个promise 

​		//reject() 用于拒绝这个决议

​	})

​	reject(…)就是拒绝这个promise，resolve(…)既可能完成promise，也可能拒绝，要根据传入参数而定。如果传给resolve(…)的是一个非Promise、非thenable的立即值，这个Promise就会用这个值完成。

​	描述：

​	Promise对象是一个代理对象(代理一个值)， 被代理的值在Promise对象创建时可能是未知的。它允许你为异步操作的成功和失败分别绑定相应的处理方法(handlers)。这让异步方法可以像同步方法那样返回值，但并不是立即返回最终执行结果，而是一个能代表未来出现的结果的promise结果

​	一个Promise有以下几种状态：

​	pending: 初始状态，不是成功或失败状态

​	fulfilled: 意味着操作成功完成

​	rejected:   意味着操作失败

pending状态的Promise对象可能触发fulfilled状态并传递一个值给相应的状态处理方法，也可能触发失败状态(rejected)并传递失败信息。当其中任一种情况出现时，Promise对象的then方法绑定的处理方法(handlers)就会调用(then方法包含两个参数：onfulfilled和onrejected，它们都是Function类型。当Promise状态为fulfilled时， 调用then的onfulfilled方法，当Promise状态为*rejected*时，调用 then 的 onrejected 方法，所以咋异步操作的完成和绑定处理方法之间不存在竞争)。

![promises](/Users/yaoxiaohui/Downloads/promises.png)

属性

Promise.length : 长度属性，其值总是为1(构造器参数的数目)

Promise.prototype: 表示Promise构造器的原型

方法

Promise.resolve(value)

​	返回一个状态由给定value决定的Promise对象。如果value是thenable(即带有then方法)， 返回的Promise采用传入的这个thenable的最终决议值，可能是完成，也可能是拒绝

```javascript
var p1 = Promise.resolve({

	then: function(onFulFill, onReject){ onFulFill("fulfilled!") }

})	
console.log(p1 instanceOf Promise) //true

p1.then(function(v){
    console.log(v);//输出"fulfilled!"
}， function(e){
    //不会被调用
})

var thenable = {
    then: function(resolve){
        throw new TypeError("Throwing");
      resolve("Resolveing")
    }
};

var p2 = Promise.resolve(thenable);
p2.then(function(v) {
    //不会被调用
}, function(e) {
    console.log(e);
})
```

