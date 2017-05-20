---
id: 471
title: Proxy design pattern in iOS
date: 2016-12-19T14:17:33+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=471
permalink: /ios/proxy-design-pattern-ios/
categories:
  - Design patterns
  - iOS
tags:
  - design patterns
  - ios
  - proxy
  - UIAppearance
---
## My thoughts on the proxy design pattern in iOS


### What this article is all about

I might have to disappoint you, but this is not going to be tutorial on using the proxy design pattern. It's going to be more like a discussion about how it is used internally in the Cocoa framework and how we can take that as example and consider that advice (or not) in our projects. So if you are just interested in what a Proxy is and how it's used... Honestly, I have no idea what you are doing here. Just [read the Wikipedia page](https://en.wikipedia.org/wiki/Proxy_pattern) or something :P

### The proxy design pattern

Just so we are on the same page on this, lets look at the definition for the proxy design pattern:

> A proxy, in its most general form, is a class functioning as an interface to something else.

This is taken right from that Wikipedia page I mentioned. Reading it, though, gives a very vague definition of the proxy design pattern. A person with a wilder imagination would say many things in programming are proxies. Hmm, and maybe he'd be right.

Anyway, I like to think of a proxy as a "wrapper" that keeps the interface from the original class. Naturally, who am I to say what a proxy is, and what not. I'm sure there are at least 5 counterexamples that prove my definition is incorrect. It's overly simplistic, but I think it's good to start with.

Mentioning wrappers... are they actually a design pattern? Turns out... they are! Check it out in [Wikipedia](https://en.wikipedia.org/wiki/Adapter_pattern)! Ok, yeah, it's just an alias for the Adapter design pattern, but *it's something!*

_I hate seeing memes in blog articles, so feel free to [open it externally](http://i2.kym-cdn.com/photos/images/newsfeed/000/114/139/tumblr_lgedv2Vtt21qf4x93o1_40020110725-22047-38imqt.jpg)_

### The problem with proxies

I have a problem with the second part of **my definition**:

> A proxy is like a wrapper that *keeps the interface from the original class*

> -- Me, one paragraph above

For simple classes, or more specifically classes with simple and short interfaces, that's fine. If you look at the example given in the Wiki article, the public interface for the class is only one method. It's extremely easy to mimic the same interface in the proxy. But what if we wanted to proxy a class that is way more complex. For instance `UIView`. We would have to copy every public method from `UIView`. That's quite a lot. And it's fragile - new methods can appear with every SDK release. And most of those copied methods would probably do the same thing - just call the real functionality. And I don't think this example is exaggerated in any way. After all, It could be often useful to proxy a large object.

One possible solution would be to extract the interface as a protocol and have the original class and the proxy conform to it. But, really, that doesn't make it better. The thing about protocols is, you still need to provide implementations for those methods. Again, there is a lot of copying going on.

**Note** _I know protocols have default implementation in Swift 3, and that's really cool, but that doesn't really solve the problem._

It seems at this point we have hit a rock and we start to understand why proxies are not so common in our coding life. Or have we?

### UIAppearance

The story about proxies ended for me for quite a while. I couldn't find a solution to the problem of the common interface. It was recently that I remembered it while working on my previous [article about app branding](http://devmonologue.com/ios/ios/app-branding/). If you check the [documentation for UIAppearance](https://developer.apple.com/reference/uikit/uiappearance), it says:

> Use the UIAppearance protocol to get the appearance proxy for a class.

right from the start. Well, what do you know - here's a prime example of the proxy design pattern. Moreover, UIAppearance proxies large, complex objects like `UILabel` and `UIView`. Basically most UI classes. So not only does it mimic the interface of a complex class, but even several of those. Now this is worth some additional investigation.

#### What kind of object is UIAppearance?

##### The experiment

We cannot just open UIAppearance's source code, but at least we can look at it's declaration. I found the first instance of UIAppearance in my code and clicked on "Jump to definition":

```swift
  public protocol UIAppearance : NSObjectProtocol {

  /* To customize the appearance of all instances of a class, send the relevant appearance modification messages to the appearance proxy for the class. For example, to modify the bar tint color for all UINavigationBar instances:
      [[UINavigationBar appearance] setBarTintColor:myColor];

      Note for iOS7: On iOS7 the tintColor property has moved to UIView, and now has special inherited behavior described in UIView.h.
      This inherited behavior can conflict with the appearance proxy, and therefore tintColor is now disallowed with the appearance proxy.
  */
  public static func appearance() -> Self
```

Hm, so it is actually a protocol. That's something, but doesn't really tell us a lot about who and how implements it. But what is that at the end of the snippet?

```swift
public static func appearance() -> Self
```

That can lead to somewhere. If you ever worked with UIAppearance, you know you typically call it using:

```swift
UILabel.appearance().<set something>
```

In this example, of course `UILabel` is only one of many classes that support UIAppearance. And if you go to UILabel's header, you'd find that it conforms to the UIAppearance protocol. Interesting! So if you call `UILabel.appearance()`... you get a `UILabel` instance back. Indeed, the line `let labelAppearance:UILabel = UILabel.appearance()` compiles fine and in runtime, we can inspect that `labelAppearance` is really an object of type `UILabel`. Additionally, it is always the same object.

Well that's a little disappointing. Does it mean that `UILabel` has a static label variable that holds all the appearance information and all set properties there are also reflected on new instances when they are created?

Next experiment - if `UILabel.appearance()` is a label itself, can we actually put it on screen and see what gets displayed:

```swift
  override func viewDidLoad() {
      super.viewDidLoad()
      // Do any additional setup after loading the view, typically from a nib.
      let labelAppearance = UILabel.appearance()
      labelAppearance.frame = CGRect(x: 20, y: 50, width: 100, height: 40)
      self.view.addSubview(labelAppearance)
  }
```

Easily enough, the code compiles and runs. But then:

```
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[NSMethodSignature getArgumentTypeAtIndex:]: index (2) out of bounds [0, 1]'
*** First throw call stack:
(
	0   CoreFoundation                      0x000000011290e34b __exceptionPreprocess + 171
	1   libobjc.A.dylib                     0x000000010fc7e21e objc_exception_throw + 48
	2   CoreFoundation                      0x00000001128cbd5c -[NSMethodSignature getArgumentTypeAtIndex:] + 204
	3   UIKit                               0x0000000110ab14af +[NSObject(UIAppearanceAdditions) _installAppearanceSwizzlesForSetter:] + 293
	4   UIKit                               0x0000000110ab517f -[_UIAppearance _beginListeningForAppearanceEventsForSetter:] + 248
	5   UIKit                               0x0000000110ab53f9 -[_UIAppearance _handleSetterInvocation:] + 188
	6   CoreFoundation                      0x0000000112893a2e ___forwarding___ + 526
	7   CoreFoundation                      0x0000000112893798 _CF_forwarding_prep_0 + 120
	8   UIKit                               0x00000001103b0d75 -[UIView(Internal) _addSubview:positioned:relativeTo:] + 584
	...
```

I guess the theory about `UILabel.appearance()` really being a label is slowly fading away. But that doesn't make the reality any less disappointing. Lets look at the stack trace.

First of all, the exception description looks like a standard array index out of bounds assertion. Already, it seems UIAppearance just keeps an array of applied styles. But how?

Continuing on, we can start looking at the method calls. On 8th place, we see a call to `addSubview`. This is no surprise since the documentation specifically says that UIAppearance styles are applied when a view is added to a window. Next, 3 to 5, seems interesting. Invocation... Swizzles...?

Invocations probably refer to NSInvocation, a class encapsulating method calls. It's a bit obscure, but if you've had to save methods along with parameters for later calls in (lets say) a dictionary, you might have had to use NSInvocation.

Swizzling, on the other hand, is something many of us know from the Objective-C runtime. Swizzling refers to the process of replacing a class's method implementation with another one, effectively changing the class in runtime.

##### The results

At this point, we can make a pretty good guess as to how UIAppearance does its magic. It collects styling calls as NSInvocation objects in an array. Then, it uses method swizzling to override setters and apply those invocations to all new instances of the styled class. To make the compiler happy, especially in Swift, the `appearance` method from the UIAppearance protocol makes sure to return an object of the same class. That way all the class' methods are accessible and statements such as:

```swift
UINavigationBar.appearance().barTintColor = themeColor
```

become possible. But in reality the underlying object is not an instance of that class, but a proxy collecting invocations, ready to apply them to any subsequent instances.

##### Next steps

We didn't need to get too deep into Cocoa in order to understand how UIAppearance was implemented. Just some fiddling with its declaration and lldb was enough. Of course, for a lot of things we had to use our imagination and make assumptions. For instance, we inferred implementation details based on method names and undeterministic behavior.

However, a more powerful investigation method that works especially good in Objective-C, is looking at the class dumps. As you probably know, Objective-C is relatively easy to decompile. So there are tools available that will generate the source code based on compiled binaries. Moreover, for built-in libraries, dumps are readily available in Github. So before those libraries and frameworks get re-written in Swift, we can take advantage.

If you want to confirm our findings about UIAppearance or you still have questions regarding its implementation, a good place to start would be UIKit's source code, and UIAppearanceProxy specifically, that you can [find here](https://github.com/BigZaphod/Chameleon/blob/master/UIKit/Classes/UIAppearanceProxy.m)

_**Thanks to Shoumikhin for suggesting that in the comment section!**_

### Takeaways

The key takeaway for me here is that there is no clean easy solution to the problem at hand. When you proxy a class, it is **your** responsibility to carry the interface over to the proxy. And if that interface is non-trivial, the practicality of this approach starts to fade away. And while the proxy design pattern in iOS is definitely possible and also somewhat endorced by the Cocoa framework, it is not a simple and elegant implementation. It heavily relies on Objective-C and its ability to "shapeshift" at runtime.

So whether or not you would use it, heavily depends on how comfortable and open to the Objective-C runtime you are. I, personally, avoid it in my code and I get really suspicious about third party frameworks using it without a **really** good reason.
