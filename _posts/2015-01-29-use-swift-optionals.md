---
id: 345
title: How to use Swift optionals
date: 2015-01-29T19:53:02+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=345
permalink: /swift/use-swift-optionals/
categories:
  - Swift
tags:
  - implicitly unwrapped optionals
  - Optionals
  - Swift
---
Last year Apple announced Swift. It came with a bundle of new features that were kind of unfamiliar to Objective-C developers. The most notable of these new concepts were optionals.

Swift optionals are a cornerstone feature in the language. They are nested deep inside Swift - basically everywhere you look. The thing is that they are not that hard to understand, but at the same time are difficult to master.

Sure, maybe you’ve learned how to use them, you might even have learned how to fix cryptic compiler errors related to them (if you have, you deserve applause). However, knowing how to use Swift optionals in “the real world” is a whole new thing.

Having written a certain amount of code, I definitely cannot say I know how to manage Swift optionals, but I have learned many things from experience. So what you are about read is not a definitive guide to using Swift optionals, but rather my take on them. I’ll be happy to get feedback from you if you think something I wrote is incorrect.

I’m writing this article because I feel that not much has been said about the proper way to use Swift optionals. It is my desire to at least start a discussion about it. So fee free to join in!

## Problems that Swift optionals solve

In order to see how we can use Swift optionals, we first need to determine what problems they solve. For a detailed overview on what they are, please refer to my [Swift optional values article](http://devmonologue.com/ios/uncategorized/swift-optional-values/).

Basically, Swift optionals replace `nil` from Objective-C. As you know, Cocoa is really tolerant when it comes to null objects. You can pass an empty pointer, even call a method on a nil object, without ever worrying that your application might crash. However, this is not always good. Consider this:

```objc
NSArray* myArray = [NSArray arrayWithObject:nil];
```

It crashed! Why? Because not every method in Cocoa can work with `nil`. Another example:

```objc
NSString* token = session.retrieveUserToken();
if (token == nil)
{
    // hmmm... now what?
}
```

The return value is `nil`. But what does that mean? Does it mean that the session object couldn’t get the token? Or maybe the token is just `nil`? And in general... is it OK for the token to be `nil`? Who knows...

The morale of the story is that there is no convention in Objective-C that distinguishes between an invalid object and no object at all.

Enforcing a convention about return values and parameters might not seem like a very big deal. However, when it comes to keeping code clean and easy to read, a little can come a long way. In my experience you really have to **make** developers write code in a certain way, otherwise they will slowly start drifting away and ignoring your rules.  

## When to use Swift optionals

This one is relatively easy. You use a Swift optional every time there is a chance that your variable might be `nil`. Also, most of the time the compiler will force you to apply this rule.

That being said, there are occasions where you might want to define a variable as optional when in reality you don’t need it to be. That can happen with parameter names. Even if you know you wont be needing an optional for the purpose of your method, you need to think about how you (and others) will be using it. Is it common for this function to be called in situations where the parameter names will be optionals? For example, a method that works with views inside a view controller will typically work with optionals since many of the ways to get a reference to a `UIView` object return an optional (IBOutlets, traversing the view hierarchy etc.). In such a method it would be really convenient to be able to pass an optional so that you don’t have to unwrap every time you call it.

## ? or !

A difficult decision when it comes to Swift optionals is choosing between normal and implicitly unwrapped optionals. As a newbie in Swift you might be tempted to use the latter in most cases just because it’s easier. In fact, many StackOverflow answers and blog posts suggest that. But that’s dangerous thing to do. Using implicitly unwrapped optionals is kind of like using:

```objc
NSArray* myArray = self.arrayFullOfWonders;
NSObject* firstWonder = [myArray objectAtIndex:0];
```

Yes, it should (theoretically) work, but you have to be REALLY certain that the array has at least one item. Otherwise, bad things will happen. It’s much better to write:

```objc
NSArray* myArray = self.arrayFullOfWonders;
NSObject* firstWonder = [myArray firstObject];
```

This code does the same exact thing as the previous. But it also doesn’t crash when the array is empty.

That’s exactly how I feel about Swift optionals. Using implicitly unwrapped optionals is a lot like `objectAtIndex:`. You have to be extremely suspicious about writing that “!”. Use it only when you are absolutely, 200% sure that your variable is not `nil`. In all other cases, your the conventional optionals with “?”.

## Designing with optionals in mind

The hardest aspect of Swift optionals is deciding how to use them in methods and classes that you are about to design and create. When using Cocoa classes, in most cases you are forced to use them in one way or another. A method either requires an optional or not. However, when you are the one designing that method, it can get a little more complex.

First of all, you need to decide whether or not your method parameters and return value should be optionals. For the return value this is easy - you return an optional whenever it is acceptable to return `nil` and a regular variable when you can guarantee that an object can be returned.

Parameter names can be a little bit trickier. It is easiest to just declare them as non-optionals so that your method body gets simpler. However, this is a silly way to go. There are cases where having a `nil` parameter is perfectly fine semantically. If your method signature does not allow them, you are missing values from your input domain. So, it really pays to stop and think about it for a second. Is it acceptable that some of my parameters are nil? This is an useful question even outside the scope of Swift optionals and it can lead to better design and spare you a few bugs later on.

Even if it’s not strictly valid for your parameter to be nil, it can still be useful to make them optional. As I hinted in one of the previous paragraphs, it can help you (and others) when using the method you are designing.

_Think about how your code will be used._ Will it be used with data that is mostly optional. If it’s so, it might prove useful to make it “optional friendly”. Instead of making the people using your code constantly unwrap optionals just to invoke your method, you might start using optionals yourself. It can make people’s code a lot simpler.

Of course, you shouldn’t make your parameters optionals just because it would be more convenient when using the method if that contradicts with its... uhm... philosophy.
If you feel like a method shouldn’t be accepting optional parameters, then just don’t do it. However, there are many situations where it doesn’t really matter.

When you’re working with views, for example, having optionals is not a big deal. Most of the things you can do with `UIView` can be also done transparently with an optional `UIView`:

```objc
controlsView?.hidden = true
```

Here, we don't really care if `controlsView` is `nil` or not. We just want to hide it. If it isn’t there (maybe it’s not yet allocated), the method call will just do nothing and execution will continue without any changes. Even if this wasn’t the case, we could have unwrapped the optional before doing anything to it:

```objc
func hideCnntrolsView(controlsView:UIView?) {
    if let controlsView = controlsView {
        controlsView.hidden = true
    }
}
```

Methods like this can work with both optionals and non-optionals without too much trouble. So if they will be used around many optional variables, it might make sense to make them “optionals-ready”.

To summarize, these are the basic rules I personally follow when making Swift optionals related decisions:

* If it is possible that I variable is `nil`, use an optional
* When using frameworks, always follow the API's convention
* Be very suspicious about implicitly unwrapped optionals. Use them only when:
    * You are ABSOLUTELY sure the variable will NEVER contain `nil`
    * For IBOutlets (if a non nil value cannot be parsed from the Xib, an exception will be thrown either way, so it is safe to assume the variable contains a real object)
    * For member variables that cannot be initialized in the constructor, but are guaranteed to receive a value elsewhere in the code
        * For example, if the variable is assigned in a view controller's `viewDidLoad` method
* Prefer non-optional parameters, but don't hesitate to use them if nil parameters are a valid use case
* You can make a parameter optional if that will make it’s usage easier as long as it does not conflict with the method’s “philosophy”.
