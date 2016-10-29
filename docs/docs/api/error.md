---
layout: api
id: error
title: .error
---


[← Back To API Reference](/docs/api-reference.html)
<div class="api-code-section"><markdown>
##.error

```js
.error([function(any error) rejectedHandler]) -> Promise
```

和[`.catch`](.)相似，但不是捕获所有类型的exception，二是捕获操作性error。
Like [`.catch`](.) but instead of catching all types of exceptions, it only catches operational errors.

*注意，“error” 意味着错误，比如对象的类型错误（“instanceof error”）-不是string，number等。查看 [a string is not an error](http://www.devthought.com/2011/12/22/a-string-is-not-an-error/).*
*Note, "errors" mean errors, as in objects that are `instanceof Error` - not strings, numbers and so on. See [a string is not an error](http://www.devthought.com/2011/12/22/a-string-is-not-an-error/).*

与[`.catch`](.)模式类似：
It is equivalent to the following [`.catch`](.) pattern:

```js
// Assumes OperationalError has been made global
//假设OperationalError已为全局
function isOperationalError(e) {
    if (e == null) return false;
    return (e instanceof OperationalError) || (e.isOperational === true);
}

// Now this bit:
.catch(isOperationalError, function(e) {
    // ...
})

// Is equivalent to:
//与以下的一样：
.error(function(e) {
    // ...
});
```

比如，如果一个promise化的函数errback节点类型的回调时报错了，错误会被[`.error`](.)捕获。 但是，如果节点类型的回调throw一个error，只有`.catch`才能捕获它。
For example, if a promisified function errbacks the node-style callback with an error, that could be caught with [`.error`](.). However if the node-style callback **throws** an error, only `.catch` would catch that.

下面的例子中，你可以处理比如JSON.parse时的`SyntaxError`和`fs`的Filesystem errors，但是让程序员错误以未处理的rejection的形式冒出。
In the following example you might want to handle just the `SyntaxError` from JSON.parse and Filesystem errors from `fs` but let programmer errors bubble as unhandled rejections:

```js
var fs = Promise.promisifyAll(require("fs"));

fs.readFileAsync("myfile.json").then(JSON.parse).then(function (json) {
    console.log("Successful json")
}).catch(SyntaxError, function (e) {
    console.error("file contains invalid json");
}).error(function (e) {
    console.error("unable to read file, because: ", e.message);
});
```
现在，因为这里没有一个catch-all处理，如果你打印`console.lag`（因为有个你没预料到的错误），你会看到：
Now, because there is no catch-all handler, if you typed `console.lag` (causes an error you don't expect), you will see:

```
Possibly unhandled TypeError: Object #<Console> has no method 'lag'
    at application.js:8:13
From previous event:
    at Object.<anonymous> (application.js:7:4)
    at Module._compile (module.js:449:26)
    at Object.Module._extensions..js (module.js:467:10)
    at Module.load (module.js:349:32)
    at Function.Module._load (module.js:305:12)
    at Function.Module.runMain (module.js:490:10)
    at startup (node.js:121:16)
    at node.js:761:3
```
*（如果你没有看见上面的- 你需要打开[long stack traces](/docs/api/promise.config.html)）*
*( If you don't get the above - you need to enable [long stack traces](/docs/api/promise.config.html) )*

如果文件包含无效的JSON：
And if the file contains invalid JSON:

```
file contains invalid json
```
如果 ‘fs’模块报出文件没有找到的错误
And if the `fs` module causes an error like file not found:

```
unable to read file, because:  ENOENT, open 'not_there.txt'
```
</markdown></div>

<div id="disqus_thread"></div>
<script type="text/javascript">
    var disqus_title = ".error";
    var disqus_shortname = "bluebirdjs";
    var disqus_identifier = "disqus-id-error";

    (function() {
        var dsq = document.createElement("script"); dsq.type = "text/javascript"; dsq.async = true;
        dsq.src = "//" + disqus_shortname + ".disqus.com/embed.js";
        (document.getElementsByTagName("head")[0] || document.getElementsByTagName("body")[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
