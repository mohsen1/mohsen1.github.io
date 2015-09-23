---
layout: post
title:  "Custom Errors in ES6 (ES2015)"
date:   2015-09-23 18:18:54
tags: [js]
---

A very quick note. With the new `class` and `extend` keywords it's now much easier to subclass `Error` constructor:


{% highlight javascript %}
class MyError extends Error {
  constructor(message) {
    super(message);
    this.message = message;
    this.name = 'MyError';
  }
}
{% endhighlight %}


There is no need for `this.stack = (new Error()).stack;` trick thanks to `super()` call.