---
layout: post
title:  How to kill child processes that spawn their own child processes in Node.js
date:   2014-12-31 00:28:39
---

If a child process in Node.js spawn their own child processes, `kill()` method will not kill the child process's own child processes. For example, if I start a process that starts it's own child processes via `child_process` module, killing that child process will not make my program to quit.

{% highlight javascript %}
var spawn = require('child_process').spawn;

var child = spawn('my-command');

child.kill();
{% endhighlight %}

The program above will not quit if `my-command` spins up some more processes.

### PID range hack
We can start child processes with `{detached: true}` option so those processes will not be attached to main process but they will go to a new group of processes. Then using `process.kill(-pid)` method on main process we can kill all processes that are in the same group of a child process with the same `pid` group. In my case, I only have one processes in this group.

{% highlight javascript %}
var spawn = require('child_process').spawn;

var child = spawn('my-command', {detached: true});

process.kill(-child.pid);
{% endhighlight %}

Please note `-` before `pid`. This converts a `pid` to a group of `pid`s for process `kill()` method.


