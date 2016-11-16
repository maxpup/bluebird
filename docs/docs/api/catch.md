---
layout: api
id: catch
title: .catch
---


[← Back To API Reference](/docs/api-reference.html)
<div class="api-code-section"><markdown>
##.catch

`.catch`是一个简便的处理promise链中error的方法，它包括两个值：
- 一个catch-all的变量，与同步的`catch(e) {`代码相似
- 一个filtered的变量（就像其他不是JavaScript语言一贯有的），这使你智能处理特殊的error。**这个变量常常很明显更安全，更常使用。
`.catch` is a convenience method for handling errors in promise chains. 
It comes in two variants 
 - A catch-all variant similar to the synchronous `catch(e) {` block. This variant is compatible with native promises. 
 - A filtered variant (like other non-JS languages typically have) that lets you only handle specific errors. **This variant is usually preferable and is significantly safer**. 

###一个关于promise异常处理的笔记
### A note on promise exception handling.

promise异常处理反映了JavaScript本地异常处理。一个同步函数`throw`与promise的rejecting相似。有个例子可以说明这个：
Promise exception handling mirrors native exception handling in JavaScript. A synchronous function `throw`ing is similar to a promise rejecting. Here is an example to illustrate it:

```js
function getItems(param) {
    try { 
        var items = getItemsSync();
        if(!items) throw new InvalidItemsError();  
    } catch(e) { 
        // 这里应该解决这个错误，或从getItemsSync返回一个报错值，或将错误抛出
        // can address the error here, either from getItemsSync returning a falsey value or throwing itself
        throw e; // need to re-throw the error unless I want it to be considered handled. 
        // 如果我想以后再考虑处理的话，需要重新抛出错误
    }
    return process(items);
}
```
榆次相似，promise是这样的：
Similarly, with promises:

```js
function getItems(param) {
    return getItemsAsync().then(items => {
        if(!items) throw new InvalidItemsError(); 
        return items;
    }).catch(e => {
        // 这里应该解决这个错误，或从getItemsSync返回一个报错值，或将错误抛出
        // 如果我想以后再考虑处理的话，需要重新抛出错误
        // can address the error here and recover from it, from getItemsAsync rejects or returns a falsey value
        throw e; // Need to rethrow unless we actually recovered, just like in the synchronous version
    }).then(process);
}
```

### Catch-all

```js
.catch(function(any error) handler) -> Promise
```
```js
.caught(function(any error) handler) -> Promise
```

这是一个catch-all异常处理，是调用这个promise的[`.then(null, handler)`](.)的捷径。任何发生在这个`.then`链中的异常都会被传至最近的`.catch`处理程序。
This is a catch-all exception handler, shortcut for calling [`.then(null, handler)`](.) on this promise. Any exception happening in a `.then`-chain will propagate to nearest `.catch` handler.

*对于早期ECMAScript版本的兼容性来说，`.caught`是[`.catch`](.)的别名。
*For compatibility with earlier ECMAScript versions, an alias `.caught` is provided for [`.catch`](.).*

### Filtered Catch 

```js
.catch(
    class ErrorClass|function(any error)|Object predicate...,
    function(any error) handler
) -> Promise
```
```js
.caught(
    class ErrorClass|function(any error)|Object predicate...,
    function(any error) handler
) -> Promise
```
这是[`.catch`](.)的扩展，更像java或者C#语言的那种catch-clauses。与手动检查`instanceof`或者`.name === "SomeError"不同，你可以具体说明一定数量的error构造器来限制使用这个catch处理程序的资格。这个catch处理是有资格的构造器第一次遇见的，也是唯一将要被调用的（不确定）。
This is an extension to [`.catch`](.) to work more like catch-clauses in languages like Java or C#. Instead of manually checking `instanceof` or `.name === "SomeError"`, you may specify a number of error constructors which are eligible for this catch handler. The catch handler that is first met that has eligible constructors specified, is the one that will be called.

Example:

```js
somePromise.then(function() {
    return a.b.c.d();
}).catch(TypeError, function(e) {
    //If it is a TypeError, will end up here because
    //it is a type error to reference property of undefined
    //如果这是一个TypeError，将会在这里结束。因为这是一个引用undefined属性的error类型。
}).catch(ReferenceError, function(e) {
    //如果一个根本没声明的话就会在这结束。
    //Will end up here if a was never declared at all
}).catch(function(e) {
    //捕获剩下的不是TypeError或者ReferenceError的error。
    //Generic catch-the rest, error wasn't TypeError nor
    //ReferenceError
});
 ```
你也可以给一个catch处理添加多个选择器。
You may also add multiple filters for a catch handler:

```js
somePromise.then(function() {
    return a.b.c.d();
}).catch(TypeError, ReferenceError, function(e) {
    //Will end up here on programmer error
    //编程的error会在这结束
}).catch(NetworkError, TimeoutError, function(e) {
    //可预料的日常网络错误在这终结
    //Will end up here on expected everyday network errors
}).catch(function(e) {
    //捕获所有没限制的错误
    //Catch any unexpected errors
});
```
对于一个你想捕获的error，你需要构造器的`.prototype`属性是`instanceof Error。
For a parameter to be considered a type of error that you want to filter, you need the constructor to have its `.prototype` property be `instanceof Error`.

最简单的这样的构造器可以像这样：
Such a constructor can be minimally created like so:

```js
function MyCustomError() {}
MyCustomError.prototype = Object.create(Error.prototype);
```

Using it:

```js
Promise.resolve().then(function() {
    throw new MyCustomError();
}).catch(MyCustomError, function(e) {
    //will end up here now
});
```
然而，如果你想要堆栈痕迹和清洁器（？）字符串输出，你应该这样做：
However if you  want stack traces and cleaner string output, then you should do:

*在Node.js和其他V8环境中，需要支持`Error.captureStackTrace`*
*in Node.js and other V8 environments, with support for `Error.captureStackTrace`*

```js
function MyCustomError(message) {
    this.message = message;
    this.name = "MyCustomError";
    Error.captureStackTrace(this, MyCustomError);
}
MyCustomError.prototype = Object.create(Error.prototype);
MyCustomError.prototype.constructor = MyCustomError;
```
使用CoffeeScript的 `class`也是这样的：
Using CoffeeScript's `class` for the same:

```coffee
class MyCustomError extends Error
  constructor: (@message) ->
    @name = "MyCustomError"
    Error.captureStackTrace(this, MyCustomError)
```
这种方法也支持基于判断的构造器，如果你传入一个判断（？）函数而不是一个error构造器，这个判断将会接受error作为一个参数，判断的结果将会用来决定是否这个error处理会被调用。
This method also supports predicate-based filters. If you pass a
predicate function instead of an error constructor, the predicate will receive
the error as an argument. The return result of the predicate will be used
determine whether the error handler should be called.

判断应该考虑到关于捕获error的非常细致的控制（？）：模式匹配、运算和实现许多其他技术的错误类型设置。
Predicates should allow for very fine grained control over caught errors:
pattern matching, error-type sets with set operations and many other techniques
can be implemented on top of them.

一个基于判断的选择器的例子：
Example of using a predicate-based filter:

```js
var Promise = require("bluebird");
var request = Promise.promisify(require("request"));

function ClientError(e) {
    return e.code >= 400 && e.code < 500;
}

request("http://www.google.com").then(function(contents) {
    console.log(contents);
}).catch(ClientError, function(e) {
   //A client error like 400 Bad Request happened
   //一个可滑动错误，比如400的错误请求
});
```
只检查属性的判断函数有一个简略的写法。你可以传入一个对象取代判断函数，它的属性会拿去匹配error对象。
Predicate functions that only check properties have a handy shorthand. In place of a predicate function, you can pass an object, and its properties will be checked against the error object for a match:

```js
fs.readFileAsync(...)
    .then(...)
    .catch({code: 'ENOENT'}, function(e) {
        console.log("file not found: " + e.path);
    });
```
`{code: 'ENOENT'}`是判断函数function predicate(e) { return isObject(e) && e.code == 'ENOENT' }`的简略写法，I.E.（？），宽松的相等会用到。
The object predicate passed to `.catch` in the above code (`{code: 'ENOENT'}`) is shorthand for a predicate function `function predicate(e) { return isObject(e) && e.code == 'ENOENT' }`, I.E. loose equality is used.

*对于早期ECMAScript版本的兼容性来说，`.caught`是[`.catch`](.)的别名。
*For compatibility with earlier ECMAScript version, an alias `.caught` is provided for [`.catch`](.).*
</markdown></div>

通过不返回一个rejected值或者从catch中抛出异常，你会“从失败中回复”，并且继续这个链式操作：
By not returning a rejected value or `throw`ing from a catch, you "recover from failure" and continue the chain:

```js
Promise.reject(Error('fail!'))
  .catch(function(e) {
    // fallback with "recover from failure"
    //失败中恢复的退路
    return Promise.resolve('success!'); // promise or value
  })
  .then(function(result) {
    console.log(result); // will print "success!"
  });
```
这正像同步的代码一样：
This is exactly like the synchronous code:

```js
var result;
try {
  throw Error('fail');
} catch(e) {
  result = 'success!';
}
console.log(result);
```

<div id="disqus_thread"></div>
<script type="text/javascript">
    var disqus_title = ".catch";
    var disqus_shortname = "bluebirdjs";
    var disqus_identifier = "disqus-id-catch";
    
    (function() {
        var dsq = document.createElement("script"); dsq.type = "text/javascript"; dsq.async = true;
        dsq.src = "//" + disqus_shortname + ".disqus.com/embed.js";
        (document.getElementsByTagName("head")[0] || document.getElementsByTagName("body")[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
