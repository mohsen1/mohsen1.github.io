---
layout: post
title:  Compare NSDate instance with ease in Swift
date:   2015-01-06 18:28:39
tags: [swift]
---

To make it easy comparing two `NSDate` instances in Swift we can overload `<=`, `>=`, `>`, `<` and `==` operators with `NSDate` types on left and right hand sides of overloading functions. `timeIntervalSince1970` is a safe measure for comparing most dates. I used `timeIntervalSince1970` to make the decision if two dates are equal, less or greater.

{% highlight swift %}
func <=(lhs: NSDate, rhs: NSDate) -> Bool {
    return lhs.timeIntervalSince1970 <= rhs.timeIntervalSince1970
}
func >=(lhs: NSDate, rhs: NSDate) -> Bool {
    return lhs.timeIntervalSince1970 >= rhs.timeIntervalSince1970
}
func >(lhs: NSDate, rhs: NSDate) -> Bool {
    return lhs.timeIntervalSince1970 > rhs.timeIntervalSince1970
}
func <(lhs: NSDate, rhs: NSDate) -> Bool {
    return lhs.timeIntervalSince1970 < rhs.timeIntervalSince1970
}
func ==(lhs: NSDate, rhs: NSDate) -> Bool {
    return lhs.timeIntervalSince1970 == rhs.timeIntervalSince1970
}
{% endhighlight %}

Note that operator overloading declarations should be placed in global context. I highly recommend documenting this behavior in your developer guide documents.

With those operator overloading declarations in place, now we can compare dates with ease:

{% highlight swift %}
let date0 = NSDate(timeIntervalSince1970: 0)
let date1 = NSDate(timeIntervalSince1970: 0)
let date2 = NSDate(timeIntervalSince1970: 1839203982)
let date3 = NSDate(timeIntervalSince1970: 1339203982)

date1 < date2 // true
date0 == date1 // true
date3 > date2 // false
{% endhighlight %}
