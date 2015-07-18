---
layout: post
title:  "How to split a Swagger spec into smaller files"
date:   2015-07-16 19:48:19
tags: [swagger]
---

If you're writing a [Swagger](http://swagger.io) API spec and it's becoming too large, you can split it into multiple files. Swagger supports [JSON Reference (draft)](https://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03) for using remote and local pieces of JSON to build up a Swagger document.

### JSON Reference Overview

JSON Reference uses the special key `$ref` to define a "reference" to a piece of JSON. For example following JSON has a reference to `http://example.com/foo.json`:


{% highlight json %}
{
  "foo": {
    "$ref": "http://example.com/foo.json"
  }
}
{% endhighlight %}

Imagine the JSON file at `http://example.com/foo.json` is following JSON:

{% highlight json %}
{
  "bar": 1
}
{% endhighlight %}

If we "resolve" the JSON object that had a `$ref` it will give us this:

{% highlight json %}
{
  "foo": {
    "bar": 1
  }
}
{% endhighlight %}

As you see object **containing the `$ref`** value is **replaced** with the object that reference was pointing to. Object containing `$ref` con not have any other property. If they did, those properties would've got lost during resolution.


### Remote And Local References

JSON References can be remote or local. A local reference, just like a local link in a HTML file starts with `#`. A local reference uses a [JSON Pointer (RFC 6901)](https://tools.ietf.org/html/rfc6901) to point to a piece of JSON inside current document. Consider following example:

{% highlight json %}
{
  "info": {
    "version": "1.0.0"
  },
  "item": {
    "information": {
      "$ref": "#/info"
    }
  }
}
{% endhighlight %}

After reference resolution, that JSON will be transformed to to this:

{% highlight json %}
{
  "info": {
    "version": "1.0.0"
  },
  "item": {
    "information": {
      "version": "1.0.0"
    }
  }
}
{% endhighlight %}

Note that `"info"` was not removed from our JSON and the key `"info"` was not added to the `"information"` object. We just replaced what is inside `"info"` object with object containing `$ref` reference. Using local `$ref` is a great way of avoiding repeating yourself when writing a JSON object.

As a convention, when defining a [JSON Schema (RFC draft)](https://tools.ietf.org/html/draft-zyp-json-schema-03) all the objects that will get repeated go to `"definitions"` object. Swagger embraces this and uses `"definitions"` object as a place to hold your API models. The API models are used in parameters, responses and other places of a Swagger spec.

### Using JSON References to split up a Swagger spec
Swagger spec can use `$ref`s anywhere in the spec. You can put a reference instead of any object in Swagger. By default Swagger encourages spec developers to put their models in `"definitions"` object. But you can do more than that and use `$ref`s to put parts of your API spec into different files. For simplicity I am going to use YAML for rest of examples.

Imagine you have a Swagger spec like this:

{% highlight yaml %}
swagger: '2.0'
info:
  version: 0.0.0
  title: Simple API
paths:
  /foo:
    get:
      responses:
        200:
          description: OK
  /bar:
    get:
      responses:
        200:
          description: OK
          schema:
            $ref: '#/definitions/User'
definitions:
  User:
    type: object
    properties:
      name:
        type: string
{% endhighlight %}

This Swagger spec is very simple but we can still split it into smaller files.

First we need to define our folder structure. Here is our desired folder structure:

{% highlight sh %}
.
├── index.yaml
├── info
│   └── index.yaml
├── definitions
│   └── index.yaml
│   └── User.yaml
└── paths
    ├── index.yaml
    ├── bar.yaml
    └── foo.yaml
{% endhighlight %}

We keep root items in `index.yaml` and put everything else in other files. Using `index.yaml` as file name for your root file is a convention. In folders that hold only one file we also use `index.yaml` as their file name.

Here is list of files with their contents:

**&nbsp;**
**`index.yaml`**
{% highlight yaml %}
swagger: '2.0'
info:
  $ref: ./info/index.yaml
paths:
  $ref: ./paths/index.yaml
definitions:
  $ref: ./definitions/index.yaml
{% endhighlight %}

**&nbsp;**
**`info/index.yaml`**
{% highlight yaml %}
version: 0.0.0
title: Simple API
{% endhighlight %}

**&nbsp;**
**`definitions/index.yaml`**
{% highlight yaml %}
User:
  $ref: ./User.yaml
{% endhighlight %}

**&nbsp;**
**`definitions/User.yaml`**
{% highlight yaml %}
type: object
properties:
  name:
    type: string
{% endhighlight %}

**&nbsp;**
**`paths/index.yaml`**
{% highlight yaml %}
/foo:
  $ref: ./foo.yaml
/bar:
  $ref: ./bar.yaml
{% endhighlight %}

**&nbsp;**
**`paths/foo.yaml`**
{% highlight yaml %}
get:
  responses:
    200:
      description: OK
{% endhighlight %}

**&nbsp;**
**`paths/bar.yaml`**
{% highlight yaml %}
get:
  responses:
    200:
      description: OK
      schema:
        $ref: '#/definitions/User'
{% endhighlight %}

Note that in `paths/bar.yaml` we are using a local reference while in the file itself the local reference will not get resolved. Most resolvers will resolve remote references first and the resolve local references. With that order, `$ref: '#/definitions/User'` will be resolved inside `index.yaml` after `definitions/User.yaml` is populated in it.


### Tools
[`json-refs`](https://github.com/whitlockjc/json-refs) is the tool for resolving a set of partial JSON files into a single file. Apigee's [Jeremy Whitlock](https://twitter.com/whitlockjc) did the hard work of writing this library.

Here is an example of how to use JSON Refs and [YAML-JS](https://github.com/nodeca/js-yaml) to resolve our multi-file Swagger:

{% highlight js %}
var resolve = require('json-refs').resolveRefs;
var YAML = require('js-yaml');
var fs = require('fs');

var root = YAML.load(fs.readFileSync('index.yaml').toString());
var options = {
  processContent: function (content) {
    return YAML.load(content);
  }
};
resolve(root, options).then(function (results) {
  console.log(YAML.dump(results.resolved));
});
{% endhighlight %}

### Repository

You can find the example in this blog post [in this GitHub repository](https://github.com/mohsen1/multi-file-swagger-example)
