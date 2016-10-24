---
id: anti-patterns
title: Anti-patterns
---

This page will contain common promise anti-patterns that are exercised in the wild.
这篇文章包含外界出现过的普遍的反模式promise。


- [The explicit construction anti-pattern](#the-explicit-construction-anti-pattern)
- [The `.then(success, fail)` anti-pattern](#the-.then)

##The Explicit Construction Anti-Pattern

这是最普遍的反模式，也叫作promise构造器反模式，还称为延迟的反模式。如果你没有真正理解promise，而是将他们当中牛逼闪闪的event emiiter或者callback，你很容易陷入这种错误模式。总的来说，promise是为了保留平坦的缩进或者一个异常通道等同步代码失去的属性而出现的异步代码。
This is the most common anti-pattern. It is easy to fall into this when you don't really understand promises and think of them as glorified event emitters or callback utility. It's also sometimes called the promise constructor anti-pattern. Let's recap: promises are about making asynchronous code retain most of the lost properties of synchronous code such as flat indentation and one exception channel. This pattern is also called the deferred anti-pattern.

在显示构造反模式中，promise表现为无理由的、复杂的代码。
In the explicit construction anti-pattern, promise objects are created for no reason, complicating code.

第一个例子是当你已经有了一个promise或者thenable还去声明一个deferred对象。
First example is creating deferred object when you already have a promise or thenable:

```js
//Code copyright by Twisternha http://stackoverflow.com/a/19486699/995876 CC BY-SA 2.5
myApp.factory('Configurations', function (Restangular, MotorRestangular, $q) {
    var getConfigurations = function () {
        var deferred = $q.defer();

        MotorRestangular.all('Motors').getList().then(function (Motors) {
            //Group by Config
            var g = _.groupBy(Motors, 'configuration');
            //Map values
            var mapped = _.map(g, function (m) {
                return {
                    id: m[0].configuration,
                    configuration: m[0].configuration,
                    sizes: _.map(m, function (a) {
                        return a.sizeMm
                    })
                }
            });
            deferred.resolve(mapped);
        });
        return deferred.promise;
    };

    return {
        config: getConfigurations()
    }

});
```

这种多余的包装非常危险，所有错误和拒绝都会被掩盖，无法传到function的调用者那里。
This superfluous wrapping is also dangerous, any kind of errors and rejections are swallowed and not propagated to the caller of this function.

取而代之的是应该简单的使用return返回已有的promise和传递的值。
Instead of using the Deferred anti-pattern, the code should simply return the promise it already has and propagate values using `return`:

```js
myApp.factory('Configurations', function (Restangular, MotorRestangular, $q) {
    var getConfigurations = function () {
        //Just return the promise we already have!
        return MotorRestangular.all('Motors').getList().then(function (Motors) {
            //Group by Cofig
            var g = _.groupBy(Motors, 'configuration');
            //Return the mapped array as the value of this promise
            return _.map(g, function (m) {
                return {
                    id: m[0].configuration,
                    configuration: m[0].configuration,
                    sizes: _.map(m, function (a) {
                        return a.sizeMm
                    })
                }
            });
        });
    };

    return {
        config: getConfigurations()
    }

});
```
不仅代码更简单，而且任何error都会被适当的传递到最后的使用者那去。
Not only is the code shorter but more importantly, if there is any error it will propagate properly to the final consumer.

第二个例子是创建了一个封装了callback API但并没撒用的函数。
Second example is creating a function that does nothing but manually wrap a callback API and doing a poor job at that:

```js
function applicationFunction(arg1) {
    return new Promise(function(resolve, reject){ //Or Q.defer() in Q
      libraryFunction(arg1, function (err, value) {
        if (err) {
          reject(err);
        } else {
          resolve(value);
        }
    });
}
```
这是一种笨办法，因为任何封装的callback可以使用promise库里的promisification方法快速的生成。
This is reinventing the square wheel because any callback API wrapping can and should be done immediately using the promise library's promisification methods:

```js
var applicationFunction = Promise.promisify(libraryFunction);
```
泛型的promisification有可能更快，因为它可以直接使用内部构件，而且能够处理边界事件，比如，‘libraryFunction’抛出一个同步异常或者使用多个success值。
The generic promisification is likely to be faster because it can use internals directly but also handles edge cases like `libraryFunction` throwing synchronously or using multiple success values.


**So when should deferred be used?**

简单来说，当你必须的时候。
Well simply, when you have to.

当你需要封装一个不服从标准传统的callback API的时候你或许必须使用一个defferred对象。
You might have to use a deferred object when wrapping a callback API that doesn't follow the standard convention. Like `setTimeout`:

```js
//setTimeout that returns a promise
function delay(ms) {
    var deferred = Promise.defer(); // warning, defer is deprecated, use the promise constructor
    setTimeout(function(){
        deferred.fulfill();
    }, ms);
    return deferred.promise;
}
```

这种封装应该很少，如果这是一个很普遍的promise库不能promise化它们的问题，你应该提出一个issue。
Such wrappers should be rare, if they're common for the reason that the promise library cannot generically promisify them, you should file an issue.

如果你不能静态promisification（promisify和promisifyAll在运行时太慢了），你可以使用
If you cannot do static promisification (promisify and promisifyAll perform too slowly to use at runtime), you may use [Promise.fromCallback](.).

Also see [this StackOverflow question](http://stackoverflow.com/questions/23803743/what-is-the-deferred-antipattern-and-how-do-i-avoid-it) for more examples and a debate around it.

##The `.then(success, fail)` anti-pattern

一个使用promise明确的标志是美化回调，
*Almost* a sure sign of using promises as glorified callbacks. Instead of `doThat(function(err, success))` you do `doThat().then(success, err)` and rationalize to yourself that at least the code is "less coupled" or something.

The `.then` signature is mostly about interop, there is *almost* never a reason to use `.then(success, fail)` in application code. It is even awkward to express it in the sync parallel:

```js
var t0;
try {
    t0 = doThat();
}
catch(e) {

}
//deal with t0 here and waste the try-catch
var stuff = JSON.parse(t0);
```

It is more likely that you would write this instead in the sync world:

```js
try {
    var stuff = JSON.parse(doThat());
}
catch(e) {

}
```

So please write the same when using promises too:

```js
doThat()
.then(function(v) {
    return JSON.parse(v);
})
.catch(function(e) {

});
```

`.catch` is specified for built-in Javascript promises and is "sugar" for `.then(null, function(){})`. Since the way errors work in promises is almost the entire point (and the only thing jQuery never got right, even if it used `.pipe` as a `.then`), I really hope the implementation you are using provides this method for readability.
