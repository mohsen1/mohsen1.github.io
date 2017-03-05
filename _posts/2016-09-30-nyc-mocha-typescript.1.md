---
layout: post
title:  "Setting up test coverage using Mocha, Istanbul, NYC with TypeScript"
date:   2016-09-30 15:28:23
tags: [js, node]
---

It's a pleasure to work with a project that uses TypeScript for your source code and tests, although setting up test coverage can be a bit tricky. I recently started a project that uses TypeScript for source as well as the tests.

I used Mocha to run the test and [`nyc`](https://github.com/istanbuljs/nyc) for generating test coverage. You will also need to `npm install --save-dev source-map-support`.

It tooks many hours to figure out a solution that works end-to-end so I wanted to share the end result.

Here is what the npm script section is looking like:

{% highlight json %}
  "scripts": {
    "test": "nyc mocha",
  },
  "nyc": {
    "include": [
      "src/**/*.ts",
      "src/**/*.tsx"
    ],
    "extension": [
      ".ts",
      ".tsx"
    ],
    "require": [
      "ts-node/register"
    ],
    "reporter": [
      "text-summary",
      "html"
    ],
    "sourceMap": true,
    "instrument": true
  },
{% endhighlight %}

Mocha configuration is located in `test/mocha.opts`:

{% highlight plain %}
--compilers ts-node/register
--require source-map-support/register
--full-trace
--bail
src/**/*.test.ts src/**/*.test.tsx
{% endhighlight %}

That's it! Now you can run `npm test` to run your tests and get coverage report.

Now all test coverage reports are mapped using sourcemaps.
