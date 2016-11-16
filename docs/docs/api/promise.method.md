---
layout: api
id: promise.method
title: Promise.method
---


[← Back To API Reference](/docs/api-reference.html)
<div class="api-code-section"><markdown>
##Promise.method

```js
Promise.method(function(...arguments) fn) -> function
```

返回一个包装了参数`fn`的新函数。新函数会返回一个promise，这个promise会跟随原来promise的状态，原来的promise返回的值fulfill它或者抛出的异常reject它。
Returns a new function that wraps the given function `fn`. The new function will always return a promise that is fulfilled with the original functions return values or rejected with thrown exceptions from the original function.

当一个函数有时会同时返回或者抛出错误的时候这种方法非常方便。
This method is convenient when a function can sometimes return synchronously or throw synchronously.

使用`Promise.method`的例子：
Example without using `Promise.method`:


```js
MyClass.prototype.method = function(input) {
    if (!this.isValid(input)) {
        return Promise.reject(new TypeError("input is not valid"));
    }

    if (this.cache(input)) {
        return Promise.resolve(this.someCachedValue);
    }

    return db.queryAsync(input).bind(this).then(function(value) {
        this.someCachedValue = value;
        return value;
    });
};
```
使用了`Promise.method`后，不需要手动将返回或抛出值包装进promise。
Using the same function `Promise.method`, there is no need to manually wrap direct return or throw values into a promise:

```js
MyClass.prototype.method = Promise.method(function(input) {
    if (!this.isValid(input)) {
        throw new TypeError("input is not valid");
    }

    if (this.cache(input)) {
        return this.someCachedValue;
    }

    return db.queryAsync(input).bind(this).then(function(value) {
        this.someCachedValue = value;
        return value;
    });
});
```
</markdown></div>

<div id="disqus_thread"></div>
<script type="text/javascript">
    var disqus_title = "Promise.method";
    var disqus_shortname = "bluebirdjs";
    var disqus_identifier = "disqus-id-promise.method";
    
    (function() {
        var dsq = document.createElement("script"); dsq.type = "text/javascript"; dsq.async = true;
        dsq.src = "//" + disqus_shortname + ".disqus.com/embed.js";
        (document.getElementsByTagName("head")[0] || document.getElementsByTagName("body")[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
