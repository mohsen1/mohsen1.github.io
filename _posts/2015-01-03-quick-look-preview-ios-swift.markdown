---
layout: post
title:  Get QuickLook Preview of Swift objects in XCode
date:   2015-01-03 00:28:39
---

When setting breakpoints in XCode, it's quite hard to see what exactly is inside an object. All XCode give you is memory address of that object. In XCode 6 it's possible to overcome this by implementing `debugQuickLookObject` method in your object. This function will be called when program is stopped by a breakpoint and you hover over the object and select the little eye icon.

For example, in my `File` class, I've implemented this method in my class. As you can see the output is very useful and handy for debugging. It works great for `NSManagedObject`s too!

![Quick look of an object in XCode](/assets/images/ios-quicklook.png)

{% highlight swift %}
class File: NSManagedObject {
    @NSManaged var id: NSNumber
    @NSManaged var parent_id: NSNumber
    @NSManaged var name: String!
    @NSManaged var content_type: String!

    func init(json:NSDictionary){ /* ... */
    }

    func debugQuickLookObject() -> AnyObject? {
        return "\(name)\ntype:\(content_type)"
    }
}
{% endhighlight %}

`debugQuickLookObject` can return almost anything. From a string to image to sounds. It should return one of the cases of [`QuickLookObject`](http://swifter.natecook.com/type/QuickLookObject/) which is listed here:

{% highlight swift %}
enum QuickLookObject {
    case Text(String)
    case Int(Int64)
    case UInt(UInt64)
    case Float(Double)
    case Image(Any)
    case Sound(Any)
    case Color(Any)
    case BezierPath(Any)
    case AttributedString(Any)
    case Rectangle(Double, Double, Double, Double)
    case Point(Double, Double)
    case Size(Double, Double)
    case Logical(Bool)
    case Range(UInt64, UInt64)
    case View(Any)
    case Sprite(Any)
    case URL(String)
}
{% endhighlight %}
