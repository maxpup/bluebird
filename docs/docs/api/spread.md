---
layout: api
id: spread
title: .spread
---


[← Back To API Reference](/docs/api-reference.html)
<div class="api-code-section"><markdown>
##.spread

```js
.spread(
    [function(any values...) fulfilledHandler]
) -> Promise
```

就像调用'.then',但是完成值必须是个数组，这个数组被传进fulfillment handler。
Like calling `.then`, but the fulfillment value _must be_ an array, which is flattened to the formal parameters of the fulfillment handler.

```js
Promise.all([
    fs.readFileAsync("file1.txt"),
    fs.readFileAsync("file2.txt")
]).spread(function(file1text, file2text) {
    if (file1text === file2text) {
        console.log("files are equal");
    }
    else {
        console.log("files are not equal");
    }
});
```
当链式调用‘.spread’时，返回一个promise的数组同样可以。
When chaining `.spread`, returning an array of promises also works:

```js
Promise.delay(500).then(function() {
   return [fs.readFileAsync("file1.txt"),
           fs.readFileAsync("file2.txt")] ;
}).spread(function(file1text, file2text) {
    if (file1text === file2text) {
        console.log("files are equal");
    }
    else {
        console.log("files are not equal");
    }
});
```
注意，如果使用ES6的时候，以上的代码需要用‘.then()’代替：
Note that if using ES6, the above can be replaced with [.then()](.) and destructuring:

```js
Promise.delay(500).then(function() {
   return [fs.readFileAsync("file1.txt"),
           fs.readFileAsync("file2.txt")] ;
}).all().then(function([file1text, file2text]) {
    if (file1text === file2text) {
        console.log("files are equal");
    }
    else {
        console.log("files are not equal");
    }
});
```

Note that [.spread()](.) implicitly does [.all()](.) but the ES6 destructuring syntax doesn't, hence the manual `.all()` call in the above code.

If you want to coordinate several discrete concurrent promises, use [`Promise.join`](.)
</markdown></div>

<div id="disqus_thread"></div>
<script type="text/javascript">
    var disqus_title = ".spread";
    var disqus_shortname = "bluebirdjs";
    var disqus_identifier = "disqus-id-spread";

    (function() {
        var dsq = document.createElement("script"); dsq.type = "text/javascript"; dsq.async = true;
        dsq.src = "//" + disqus_shortname + ".disqus.com/embed.js";
        (document.getElementsByTagName("head")[0] || document.getElementsByTagName("body")[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
