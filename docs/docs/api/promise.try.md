---
layout: api
id: promise.try
title: Promise.try
---


[← Back To API Reference](/docs/api-reference.html)
<div class="api-code-section"><markdown>
##Promise.try

```js
Promise.try(function() fn) -> Promise
```
```js
Promise.attempt(function() fn) -> Promise
```

使用`Promise.try`开始一个promise链。任何同步的异常都会rejected到返回的promise上去。
Start the chain of promises with `Promise.try`. Any synchronous exceptions will be turned into rejections on the returned promise.

```js
function getUserById(id) {
    return Promise.try(function() {
        if (typeof id !== "number") {
            throw new Error("id must be a number");
        }
        return db.getUserById(id);
    });
}
```
如果有人使用这个函数，他们会在Promise的`.catch处理函数中捕获所有的error，而不需要去处理同步和异步的异常流。
Now if someone uses this function, they will catch all errors in their Promise `.catch` handlers instead of having to handle both synchronous and asynchronous exception flows.

*为了兼容早期的ECMAScript版本，`Promise.attempt`作为别名提供给[`Promise.try`](.).*
*For compatibility with earlier ECMAScript version, an alias `Promise.attempt` is provided for [`Promise.try`](.).*
</markdown></div>

<div id="disqus_thread"></div>
<script type="text/javascript">
    var disqus_title = "Promise.try";
    var disqus_shortname = "bluebirdjs";
    var disqus_identifier = "disqus-id-promise.try";
    
    (function() {
        var dsq = document.createElement("script"); dsq.type = "text/javascript"; dsq.async = true;
        dsq.src = "//" + disqus_shortname + ".disqus.com/embed.js";
        (document.getElementsByTagName("head")[0] || document.getElementsByTagName("body")[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
