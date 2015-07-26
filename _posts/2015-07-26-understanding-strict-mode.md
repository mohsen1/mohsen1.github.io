---
layout: post
title:  "Understanding JavaScript Backward compability Strict Mode"
date:   2015-07-26 11:28:23
tags: [js, node]
---

When I started reading about the new version of JavaScript (ES6 or ES2015) I was conflicted about how adding new keywords to the language will not break JavaScript backward compability. As the only runtime language of the Web JavaScript have to be backward compatible. Any code that is valid and running today should be working in feature JavaScript engines.

Here is the list of all JavaScript reserved keywords in ES5 [§7.6.1.1](http://www.ecma-international.org/publications/files/ECMA-ST-ARCH/ECMA-262%205th%20edition%20December%202009.pdf):

```
break       do        instanceof      typeof       case         else
new         var       catch           finally      return       void
continue    for       switch          while        debugger     function
this        with      default         if           throw        delete
in          try 
```

It's illegal to use a reserved keyword as a vriable or function name in JavaScript. 

```js
var delete = new Button('delete');
// Throws SyntaxError: Cannot use the keyword 'delete' as a variable name.
```

But it's perfectly fine to use ES6 resevered words like `let` as a variable name

```js
var let = 1;
// No error
```

ES5 defines this set of keywords as ["FutureReservedWord"](https://es5.github.io/#C):

```
implements     interface   let       package    private
protected      public      static    yield
```

The *FutureReservedWord* keywords are not enforced in non-strict JavaScript. But in strict mode they are considered reserved words and it's illegal to use them as variable names.

```
(function(){
  "use strict"
  var let = 1;
})();
// Throws SyntaxError: Cannot use the reserved word 'let' as a variable name in strict mode.
```

The strict mode helps JavaScript engines determine which set of reserved words to enforce without breaking backward compability.

### ES6 code without `"use strict"`
One might ask what happens if there is no `"use strict"` in the code but it's using ES6 features like generator functions? Which set of reserved words will be enforced?

example:

```js
var let = function* (){
  yield 1;
}
```

JavaScript engines will use [ES6 set of reserverd words](http://ecma-international.org/ecma-262/5.1/#sec-7.6.1.1) and FutureReservedWord as soon as they see a ES6 specific feature like generator function. Our example will throw an `SyntaxError`.

