---
layout: post
title:  "Understanding JavaScript Backward Compatibility And The Strict Mode"
date:   2016-02-11 11:28:23
tags: [js, node]
---

When I started reading about the new version of JavaScript (ES6 or ES2015) I was conflicted about how adding new keywords to the language will not break JavaScript backward compatibility. As the only runtime language of the Web, JavaScript has to be backward compatible. Any code that is valid and running today should be working in feature JavaScript engines.

Here is the list of all JavaScript reserved keywords in **ES5** [§7.6.1.1](http://www.ecma-international.org/publications/files/ECMA-ST-ARCH/ECMA-262%205th%20edition%20December%202009.pdf):

{% highlight plain %}
break       do        instanceof      typeof       case         else
new         var       catch           finally      return       void
continue    for       switch          while        debugger     function
this        with      default         if           throw        delete
in          try
{% endhighlight %}

It's illegal to use a reserved keyword as a variable or function name in JavaScript. For example following code throws a `SyntaxError` which says "Cannot use the keyword 'delete' as a variable name".

{% highlight javascript %}
var delete = new document.getElementById('delete');
// Throws SyntaxError
{% endhighlight %}

But it's perfectly fine to use ES6 reserved words like `let` as a variable name in ES5.

{% highlight javascript %}
var let = 1;
// No error
{% endhighlight %}

One might wonder how this code should work in ES6 runtime since `let` is a reserved work and can not be used as a variable name.

ES5 defines two "modes" for the language. The regular JavaScript that existed before introduction of ES5 is considered "sloppy mode" and since ES5, JavaScript programmers can choose to write their program in the "strict mode". The strict mode introduces a set of new rules to JavaScript including the additional reserved words. This set of keywords is called ["FutureReservedWord"](https://es5.github.io/#C). Here is the list:

{% highlight plain %}
implements     interface   let       package    private
protected      public      static    yield
{% endhighlight %}

The *FutureReservedWord* keywords are not enforced in non-strict JavaScript. But in strict mode they are considered reserved words and it's illegal to use them as variable names.

{% highlight javascript %}
// sloppy
(function(){
  var let = 1;
})();
// No error

// strict
(function(){
  "use strict"
  var let = 1;
})();
// Throws SyntaxError: Cannot use the reserved word 'let' as a variable name in strict mode.
{% endhighlight %}

The strict mode helps JavaScript engines determine which set of reserved words to enforce without breaking backward compatibility.

### Sloppy ES6
Even when using ES6 new features like arrow functions or spreading it's legal to use FutureReservedWords as variable names:

{% highlight javascript %}
let arr = ['one', 'two', 'three'];

var [, ...let] = arr;

// let is ['two', 'three']
{% endhighlight %}


{% highlight javascript %}
const f = ()=> { var private = true; return private; }
f(); // no error
{% endhighlight %}

