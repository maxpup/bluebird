---
layout: api
id: finally
title: .finally
---


[← Back To API Reference](/docs/api-reference.html)
<div class="api-code-section"><markdown>
##.finally

```js
.finally(function() handler) -> Promise
```
```js
.lastly(function() handler) -> Promise
```

无论promise的命运如何，这个处理函数都会被调用。从这个promise返回一个新的链式promise。在最终值不能从处理函数修改的时候[`.finally`](.)有特殊的语义。
Pass a handler that will be called regardless of this promise's fate. Returns a new promise chained from this promise. There are special semantics for [`.finally`](.) in that the final value cannot be modified from the handler.

*注意：在资源管理使用[`.finally`](.)时有更好的选择。
*Note: using [`.finally`](.) for resource management has better alternatives, see [resource management](/docs/api/resource-management.html)*

看看这个例子：
Consider the example:

```js
function anyway() {
    $("#ajax-loader-animation").hide();
}

function ajaxGetAsync(url) {
    return new Promise(function (resolve, reject) {
        var xhr = new XMLHttpRequest;
        xhr.addEventListener("error", reject);
        xhr.addEventListener("load", resolve);
        xhr.open("GET", url);
        xhr.send(null);
    }).then(anyway, anyway);
}
```
这个example不会如预想一样工作，因为‘then’处理函数实际上会隐藏exception，并会在后面的链式反应中返回‘undefined’。
This example doesn't work as intended because the `then` handler actually swallows the exception and returns `undefined` for any further chainers.

这可以使用`.finally`解决。
The situation can be fixed with `.finally`:

```js
function ajaxGetAsync(url) {
    return new Promise(function (resolve, reject) {
        var xhr = new XMLHttpRequest;
        xhr.addEventListener("error", reject);
        xhr.addEventListener("load", resolve);
        xhr.open("GET", url);
        xhr.send(null);
    }).finally(function() {
        $("#ajax-loader-animation").hide();
    });
}
```
现在动画是隐藏了，但是，除非抛出一个异常，这个函数对于返回promise的fulfilled或者rejected值没有效果。这和同步时‘finnally’主题词的表现差不多。
Now the animation is hidden but, unless it throws an exception, the function has no effect on the fulfilled or rejected value of the returned promise.  This is similar to how the synchronous `finally` keyword behaves.

如果传入`.finally`的处理函数返回一个promise，直到处理函数返回的promise解决了，‘.finally’返回的函数才能被解决。如果处理函数fufill了它的promise，返回的promise会被原有的值fulfill或者reject。如果处理函数reject它的promise，返回的promise会被处理函数的值reject掉。这和在同步的‘finnally’block抛出的异常很像，都会导致原有的值或者exception忘记。如果处理函数的动作需要异步进行，这种延时很有用。比如：
If the handler function passed to `.finally` returns a promise, the promise returned by `.finally` will not be settled until the promise returned by the handler is settled.  If the handler fulfills its promise, the returned promise will be fulfilled or rejected with the original value.  If the handler rejects its promise, the returned promise will be rejected with the handler's value.  This is similar to throwing an exception in a synchronous `finally` block, causing the original value or exception to be forgotten.  This delay can be useful if the actions performed by the handler are done asynchronously.  For example:

```js
function ajaxGetAsync(url) {
    return new Promise(function (resolve, reject) {
        var xhr = new XMLHttpRequest;
        xhr.addEventListener("error", reject);
        xhr.addEventListener("load", resolve);
        xhr.open("GET", url);
        xhr.send(null);
    }).finally(function() {
        return Promise.fromCallback(function(callback) {
            $("#ajax-loader-animation").fadeOut(1000, callback);
        });
    });
}
```
如果这个渐隐的动画成功完成，返回的promise会被来自‘xhr’的值fulfilled或rejected。如果‘.fadeOut’抛出一个异常或者向callback传进一个error，从‘.fadeOut’的error会rejected返回的promise
If the fade out completes successfully, the returned promise will be fulfilled or rejected with the value from `xhr`.  If `.fadeOut` throws an exception or passes an error to the callback, the returned promise will be rejected with the error from `.fadeOut`.

*为了兼容ECMAScript版本，使用`.lastly`代替[`.finally`](.).*
*For compatibility with earlier ECMAScript version, an alias `.lastly` is provided for [`.finally`](.).*
</markdown></div>

<div id="disqus_thread"></div>
<script type="text/javascript">
    var disqus_title = ".finally";
    var disqus_shortname = "bluebirdjs";
    var disqus_identifier = "disqus-id-finally";
    
    (function() {
        var dsq = document.createElement("script"); dsq.type = "text/javascript"; dsq.async = true;
        dsq.src = "//" + disqus_shortname + ".disqus.com/embed.js";
        (document.getElementsByTagName("head")[0] || document.getElementsByTagName("body")[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
