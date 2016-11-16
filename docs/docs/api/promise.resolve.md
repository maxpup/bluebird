---
layout: api
id: promise.resolve
title: Promise.resolve
---


[← Back To API Reference](/docs/api-reference.html)
<div class="api-code-section"><markdown>
##Promise.resolve

```js
Promise.resolve(Promise<any>|any value) -> Promise
```

创造一个被给定参数决定的promise。如果这个参数已经是一个可信的`Promise`，就像它一样返回。如果这个参数不是thenable，则这个fulfilled promise会将这个参数作为完成值返回。如果这个值是thenable（像promise的对象，还有jQuery的 `$.ajax`这种行为），则返回一个吸收thenable状态的可信的promise。
Create a promise that is resolved with the given value. If `value` is already a trusted `Promise`, it is returned as is. If `value` is not a thenable, a fulfilled Promise is returned with `value` as its fulfillment value. If `value` is a thenable (Promise-like object, like those returned by jQuery's `$.ajax`), returns a trusted Promise that assimilates the state of the thenable.

如果一个函数返回一个promise但有可能返回一个静态值得时候会有用。比如，一个延迟加载的值：
This can be useful if a function returns a promise (say into a chain) but can optionally return a static value. Say, for a lazy-loaded value. Example:

```js
var someCachedValue;

var getValue = function() {
    if (someCachedValue) {
        return Promise.resolve(someCachedValue);
    }

    return db.queryAsync().then(function(value) {
        someCachedValue = value;
        return value;
    });
};
```

处理jQuery对象的例子：
Another example with handling jQuery castable objects (`$` is jQuery)

```js
Promise.resolve($.get("http://www.google.com")).then(function() {
    //Returning a thenable from a handler is automatically
    //自动返回一个来自处理函数的thenable
    //转换为一个像Promises/A规格的可信的promise
    //cast to a trusted Promise as per Promises/A+ specification
    return $.post("http://www.yahoo.com");
}).then(function() {

}).catch(function(e) {
    //jQuery不抛出实际的错误所以使用catch-all
    //jQuery doesn't throw real errors so use catch-all
    console.log(e.statusText);
});
```
</markdown></div>

<div id="disqus_thread"></div>
<script type="text/javascript">
    var disqus_title = "Promise.resolve";
    var disqus_shortname = "bluebirdjs";
    var disqus_identifier = "disqus-id-promise.resolve";
    
    (function() {
        var dsq = document.createElement("script"); dsq.type = "text/javascript"; dsq.async = true;
        dsq.src = "//" + disqus_shortname + ".disqus.com/embed.js";
        (document.getElementsByTagName("head")[0] || document.getElementsByTagName("body")[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
