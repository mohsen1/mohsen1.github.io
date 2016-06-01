---
layout: post
title:  "Non-blocking Asynchronous JSON.parse Using The Fetch API"
date:   2015-07-30 21:48:19
tags: [swagger]
---

### Update (June 2016)
Right after I published this blog post I received [this response](https://github.com/nodejs/node/issues/2031#issuecomment-126840751) from amazing Node.js developer
Vladimir Kurchatkin that JSON parsing is not happening in a different thread and in fact it is blocking the main thread. In [this tweet](https://twitter.com/mohsen____/status/628634836710756352)
I admited I was wrong and I need to update my post.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/j5bot">@j5bot</a> I need to update that post. It&#39;s not really in a background thread. It&#39;s doing the process in the next tick.<a href="https://t.co/5WMeTWb79K">https://t.co/5WMeTWb79K</a></p>&mdash; Mohsen Azimi (@mohsen____) <a href="https://twitter.com/mohsen____/status/628634836710756352">August 4, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

 [Nolan Lawson](https://twitter.com/nolanlawson) made following video to demonstrate the effect in multiple browsers:

<iframe width="560" height="315" src="https://www.youtube.com/embed/Ls8zZ4n5DUE" frameborder="0" allowfullscreen></iframe>


### The problem

I am working on [Swagger Editor](https://github.com/swagger-api/swagger-editor) performance. One of the solutions to speed things up was moving process-intensive task to Web Workers. Web Workers do a great job of moving process-heavy tasks out of the main thread but the way we can communicate with them is very slow. For each message to be sent to or received from a worker we need to convert it to a string. This means for transferring objects between the main thread and worker threads we need to `JSON.parse` and `JSON.stringify` our objects back and forth.

For larger objects, this can lead to large blocking `JSON.parse` calls. For example, when transferring back the AST from our AST-composer worker I saw a **`50ms`** pause. A 50 millisecond pause can easily drop 4 frames.
<p>
  <img src="/assets/images/slow-parse.png">
</p>

### The solution

[It's 2015 but JavaScript or the web does not have a non-blocking JSON API](https://www.reddit.com/r/javascript/comments/2uc7gv/its_2015_why_the_hell_is_jsonparse_synchronous/)! So there is no native or out of the box solution to this. Because communicating with a working is via string, doing `JSON.parse` in a worker is also pointless.

When I was exploring [the Fetch API (`window.fetch`)](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) I noticed the [`Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response) object has an asynchronous `.json` method. This is how it's used:

{% highlight js %}
fetch('/foo.json')
  .then(function(response) {
    response.json().then(function(result) {
      // result is parsed body of foo.json
    });
  });
{% endhighlight %}

We can use (abuse?) this API to move all of our JSON-parsing business out of the main thread. It can be done as simple as:

{% highlight js %}
function asyncParse(string) {
  return (new Response(string)).json();
}
{% endhighlight %}

It works as expected:

{% highlight js %}
asyncParse('{"foo": 1}').then(function (result) {
  // result is {foo: 1}
});
{% endhighlight %}

### Performance
Moving `JSON.parse` out of the main thread make the actual parsing time less important but let's see how it's different than native `JSON.parse`:

{% highlight js %}
// jsonStr is 65,183 charctars

console.time('sync: total time (blocking)');
JSON.parse(jsonStr);
console.timeEnd('sync: total time (blocking)');

console.time('async: blocking time');
console.time('async: total time');
asyncParse(jsonStr).then(function(result) {
    console.timeEnd('async: total time');
});
console.timeEnd('async: blocking time');
{% endhighlight %}

#### Result:

{% highlight python %}
sync: total time (blocking): 1.149ms
async: blocking time: 0.745ms
async: total time: 3.232ms

{% endhighlight %}

The async method is about 2x slower but hey, it's async and using it blocked the UI for less than a millisecond!

### Conclusion
I'll experiment with this and if it made sense I'll make a package and publish it. I hope JavaScript or DOM provides native non-blocking JSON APIs so we don't have to do hacks like this. With `async/await` in ES7(ES2016) working with async methods are much easier so we should have async JSON APIs as well.

