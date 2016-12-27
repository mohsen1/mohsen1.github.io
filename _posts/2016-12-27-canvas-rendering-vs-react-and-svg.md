---
layout: post
title:  "\"Use the platform\" is not always the best advise"
date:   2016-12-27 13:00:00
tags: [js]
---

Swizec Teller wrote about how different DOM diffing libraries perform under pressure in his interesting [blog post](https://swizec.com/blog/animating-svg-nodes-react-preact-inferno-vue/swizec/7311).

It's interesting to see how Preact and Vue perform better than React in that demo. In my opinion the biggest performance bottleneck is "the platform" not the libraries. I rewrote the demo to render everything in a `canvas` element. It resulted in a much smoother user interactions and animations.

<img src="/assets/images/canvas-vs-svg-react.gif" />

[**You can see the demo here**](http://azimi.me/react-fractals/) [[Source](https://github.com/mohsen1/react-fractals)]

It's important to point out that the canvas demo **does not** throttle the mouse move. It's also not using `requestAnimationFrame`.

It makes sense to use SVG for programs like this. It's more declarative and easier to work with, but as you can see in the demo, it's not fast enough.

React and other DOM diffing libraries are trying to make it easier for browser to render the demo by reducing number of updates.
The React implementation is also using mouse move event throttling and `requestAnimationFrame` to reduce number of updates.
But browsers have to consider so many different aspects of web platform while applying the update that it makes it very hard for them to be spec compliant and perform well at the same time.

Web platform is full of amazing features. It's very easy to make good looking apps using CSS and HTML. Simple things like `text-shadow` would be really hard to implement if the only API for rendering apps was just `canvas` APIs.

I'm not a browser engineer and don't know how browsers work. I'm making this assumption that when browser receives a DOM update from JavaScript it has to go through so many hoops to make sure everything is rendering per spec. While `canvas` is a very low level API that lets you draw whatever you want on. The low level API makes `canvas` fast.

Sometimes "use the platform" is not good advise because platform is polluted with tons of feature that you might not know about but browser has to take them into account.

#### Canvas DOM vs. iOS Core Graphics and UIKit
HTML canvas element API is similar to Apple Core Graphics library and DOM is similar to Apple's UIKit. I've done a little bit iOS development and learned that any UIKit component is using Core Graphics library to draw pixels. iOS developers can make custom components that uses Core Graphics library or override parts of UIKit components using Core Graphics API. It's very powerful that developers can mix and match low level and high level APIs to achieve their goals.
Web as a platform does not allow easy access to pixel drawing and layout computation algorithms. The default layout system is always enabled. It's not possible to tell browsers how to lay things on the page in a programmatic way. It's also not possible to have web components that render their own pixels without breaking web accessibility.

This is a bigger problem than just performance. Web APIs are usually very limited for doing custom behaviors. A lot of smart people tried to solve this problem. For reference [React-Canvas](https://github.com/Flipboard/react-canvas) was an attempt to solve this problem. But it can't be done without changing the platform itself.

Good news is that [CSS Paint API spec](https://drafts.css-houdini.org/css-paint-api/) is under development. It will make it possible for web components draw their pixels directly.
Another good news is [CSS Layout API spec](https://drafts.css-houdini.org/css-layout-api/) that allows developers to override default layout algorithm. These APIs are part of [Project Houdini](https://github.com/w3c/css-houdini-drafts) that is an effort for bringing low level APIs to web developers. With low level APIs developers don't have to choose between rendering everything in `canvas` or relying on web platform that's not completely under their control.

I'm very excited about project Houdini and can't wait to write apps that utilize those APIs!


