---
layout: post
title:  "High Performance Recursive HTML/JavaScript Components"
date:   2015-08-29 21:28:09
tags: [js]
---

I developed [`json-formatter`](https://github.com/mohsen1/json-formatter) directive for use in [Swagger Editor](https://github.com/swagger-api/swagger-editor). It's a simple component for rendering JSON in HTML nicely. Inspired by how WebKit renders JSON in developer tools.

{% highlight html %}
<json-formatter>
    <!-- This will result in a infinite loop -->
    <json-formatter ng-repeat="key in keys"></json-formatter>
</json-formatter>
{% endhighlight %}

It's an AngularJS directives, so you can't use simple recursion by just repeating the directive in directive template, instead you have to do some tricks to get around it. [This StackOverflow answer](http://stackoverflow.com/questions/14430655/recursion-in-angular-directives/18609594#18609594) elegantly defines a factory that overrides AngularJS' `compile` method to allow recursive directives.

When rendering large JSON objects, this directive was responding very slowly. For example for a [large JSON](https://raw.githubusercontent.com/mohsen1/json-formatter-js/ecdfa398655469c5265503fc1d267fae0c1a1800/demo/giant.json) file which would result in 24,453 HTML nodes it would take **3.34 seconds** to render. It's a lot of time for rendering ~25K nodes. Take a look at the [HAR file](/assets/other/json-formatter-angular.har)[^har].

<p>
  <img src="/assets/images/angular-timeline.png" alt="AngularJS recursive $digest calls">
</p>

AngularJS groups `$digest` calls to minimize DOM manipulations and triggering change events but with our recursive helper factory we're avoiding that optimization and running a lot of `$digest`s. That's why we end up with such slow component.

Since AngularJS has no good way of building recursive components I went ahead and rebuilt `json-formatter` in pure JavaScript. It's available [here](https://github.com/mohsen1/json-formatter-js). This component uses no framework and everything is in plain JavaScript. Recursion happens in a simple `for` loop. It's much faster compared to AngularJS directive. Look at [HAR file](/assets/other/json-formatter-js.har), the same JSON renders in **981 milliseconds**.

<p>
  <img src="/assets/images/js-timeline.png" alt="AngularJS recursive $digest calls">
</p>

## Further optimization
Our component is appending new children to parent DOM node and installing separate event listeners for each child. This is an artifact of porting the code from AngularJS to pure JavaScript. We should really do the iteration in template level without any DOM manipulation and use event delegation to have only one click event listener for entire component.

Even though without those optimization we're getting a 3X performance boost I will redactor this component with above ideas to perform better.

[^har]: Open HAR files in Timeline of Chrome Developer Tools by right clicking and selecting "Load Timeline Data"
