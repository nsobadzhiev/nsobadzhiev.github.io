---
id: 250
title: 'Swift initializers - designated and convenience'
date: 2014-07-12T18:08:55+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=250
permalink: /swift/swift-initializers-designated-convenience/
categories:
  - Swift
  - Swift vs Objective-C
tags:
  - designated
  - init
  - ios
  - secondary
  - Swift
  - Swift vs Objective-C
---
_This post is part of a [Swift vs Objective-C series][Swift vs Objective-C index] where we look at new features in Swift and how they compare to their predecesors from Objective-C.
In this article, we are going to discuss designated and convenience Swift initializers and their Objective-C counterparts._

# Designated and convenience initializers

You should already know that Objective-C has things called designated initializers. It's not something you absolutely MUST know, but it can often help you write better class interfaces.
The concept is simple - a designated initializer is guaranteed to be called during object creation. Even if you explicitly use another init method, the class "promises" to call a designated one sooner or later (a class can have more than one designated initializers and Objective-C does not specify which one should be used - it assumes all of them can fully initialize an instance). That way, you can be certain that the code in it is always going to execute no matter which constructor you use.

A convenience initializer is every initializer that is not designated. You can use them to give the clients of your class different options when it comes to creating objects. For instance, a `UIViewController` can be created:

* without any parameters
* using NSCoding (`initWithCoder:`)
* with a Nib name

Among these options, the last one (`initWithNibName:bundle:`) is the (only) designated initializer. It is quaranteed to be called during initialization.

**Note:** All Cocoa classes have a designated initializer and respect the initializer chain.

Some considerations should be taken into account with subclassing regarding the initialization chain. Basically, in addition to having a designated init method yourself, you have to make sure that it calls the corresponding designated method in it's superclass. This ensures that all classes in the inheritance tree have a chance to fully prepare the new instance and not leave any if it's variables in an undefined state.

This might not sound too important for you right now, but not respecting the designated initializer chain can lead to subtle bugs. I strongly recommend that you read the following [article about Objective C Initializer Patterns in Twitter's blog][Twitter article about initializers]

# Swift initializers vs. Objective-C init

The problem with designated initializers in Objective-C is that most developers ignore them. Unfortunately, this trend will probably continue with Swift initializers as well. You can program all your life without ever thinking about designated and secondary init methods. However, Swift initializers are a little more explicit as to what type an init method is.
Every method that you write with the following synthax:

	init (/* arguments */)

is a designated initializer. It implicitely becomes one. There is not special keyword to mark it as such. This is not true for convenience initializers. Here's an example:

	convenience init (/* arguments */)

By including the `convenience` keyword, we explicitely specified that this is a convenience initializer.

Convenience Swift initializers are subject to the following restrictions:

* They must always call a designated initializer within their body.
* A class must have at least one designated initializer, though it is possible for it to be inherited from a superclass.

The following example showcases the main use cases for convenience Swift initializers:

	class BaseClass {
    	var someVar: String

    	// a designated initializer
    	init () {
        	someVar = "test"
    	}

    	// another designated initalizer
    	init (someValue: String) {
        	someVar = someValue
    	}

    	// a convenience initializer
    	convenience init(someInt: Int) {
        	// must call self.init
       		self.init(someValue: "Hello")
    	}
	}

	// FirstSubclass does not have a designated initializer
	// It inherits it from Base Class

	class FirstSubclass : BaseClass {

    	convenience init(unsedVar: AnyObject) {
        	self.init(someValue: "First")
    	}
	}

	// SecondSubclass does not have a superclass with a designated initializer
	// thus the following code with raise a compilation error - you cannot have
	// a class without a designated initializer

	class SecondSubclass {
    	convenience init(unsedVar: AnyObject) {

    	}
	}

	// Raises a "Super.init isn't called before returning from initializer" compilation
	// error. When subclassing, all designated initializers must call one of the base
	// class's designated init methods

	class ThirdSubclass : BaseClass {
    	var data: NSData

    	init(someData: NSData) {
        	data = someData
    	}
	}

Overall, Swift initializers are a little more... pretentious when it comes to the initializing chain. While still keeping out of the way and working seamlessly with code that does not define designated and convenience init methods, it keeps reminding you to think about the initialization process. Hopefully, it will motivate us to write better class interfaces.

Don't hasitate you share your thoughts about designated and convenience Swift initializers in the comment section.
Thanks for reading!

[Swift vs Objective-C index]: http://devmonologue.com/ios/swift/swift-vs-objective-c/
[Twitter article about initializers]: https://blog.twitter.com/2014/how-to-objective-c-initializer-patterns
