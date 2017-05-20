---
id: 229
title: Swift optional values
date: 2014-06-17T19:44:11+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=229
permalink: /uncategorized/swift-optional-values/
categories:
  - Swift
  - Swift vs Objective-C
  - Uncategorized
tags:
  - ios
  - optional chaining
  - Optional values
  - Optionals
  - Swift
  - unwrapping
---
_This post is part of a [Swift vs Objective-C series][Swift vs Objective-C index] where we look at new features in Swift and how they compare to their predecesors from Objective-C._

# What are Swift optional values?

Optional values are a major weapon in Swift's arcenal. It's unlike anything I've seen in my professional carrier as a developer, though I've read that there are languages with similar concepts. They are designed as a replacement to Objective-C's way of treating `nil` references.

Swift makes heavy use of optional values. They are absolutely everywhere. So you better get used to them. The bad news is that they take a little bit of time and practice to get right. In fact, I still have problems with them at times.

# Objective-C message dispatch

Let me step back a little bit. Even the most inexperienced iOS developer has noticed that Objective-C deals with pointers to nil differently than Java for instance. For one thing, it doesn't make the application throw an exception. But why is that? And how?

The second question has a lot to do with the way Objective-C dispatches method calls to it's objects. In short, it reads the object that the reference is pointing to, and asks it which class it is a member of. Having that information, it finds the code for the right method, based to that class and if everything goes well, calls it. This mechanism is not too strict when it comes to nil pointers. When you try to dispatch a method call using such a pointer, instead of treating it was an exceptional behavior, it fails gracefully by ignoring the invokation.

One might argue that allowing that kind of behavior might result in obscure bugs, just because your methods calls might not actually work. Isn't it better to just crash and have the developer rethink his code? Well, not necessarilly. Nil pointers DO exist and they are not always signs of incorrect bahavior. Sometimes, a function has nothing to return and it has no choice but providing nil. Does this mean, the application cannot recover? Not at all. If it crashes when you try using a pointer like that, you code will soon be filled with pointless if statements that do nothing but confuse people reading it.

By allowing messages to be dispatched to nil objects, Objective-C code has much less security checks and is, therefore, much easier to read.

# Where Objective-C falls short

The previous paragraph has probably made you think that Objective-C is already good enough at handling nil references. However, there are a few exceptions. 

First of all, it only works for objects. Values, such as integers, enums and structures are not subject to the same message dispatch mechanism and, hence, don't behave that way. That means that you have to be aware what type of data you are working with.

Secondly, Objective-C's synthax does not provide any hint about the validity of returned data. You can never be sure if a method returns nil because an error has occured, or there was simply no data there to supply. You have to rely on your documentation (if you have any) or method name to let others know what it means for that method to return nil. 

# Nil 2.0 - Swift optional values

As already mentioned, Swift optional values are everywhere. I doubt there was more than a few classes that don't have them. They are the language's way of indicating that a variable does not have a value. Normally, a variable or constant points to a piece of data in memory. This is often referred to as **binding**. A variable is bound to an object. However, what happens when you have a variable, but you don't have an object to associate it with. A common example of this scenario is a function that has a return type, but given it's input, there is no data to actually provide. It still has to return something, so a common solution is to get a reference that points to an invalid address or a value that is not acceptable in the context. In Objective-C, methods return `nil` for objects and special constants like `NSNotFound` for primitive data types.

Swift handles this situation by the defining the term **no value**. Swift optional values tell the developer "This variable might have some kind of value, or no value at all". The "No value" is universally accepted across classes - "No value" for String variables is the same as "No value" for integers or view controllers. That's how Swift developers have a clear definition of what it means for a variable to **not** have a valid value.

# Swift optional values synthax

Let's compare a normal variable to a swift optional value definition.

	var myVariable: String
	
This is a standard string variable. And this:

	var myOptionalVariable: String?
	
is an optional string variable. The difference is the question mark at the end. It had to follow the data type. This definition creates the `myOptionalVariable` variable and associates it with "No value". Note that "No value" is different than nil in Objective, where it is just a pointer that refers to an invalid location.

You can explicitely specify that you want a variable to have no value by writing:

	myOptionalVariable = nil
	
## Accessing Swift optional values

Swift really wants you be aware when you're using optional values - that's why it wants you to be very explicit every time you use them. It wants to contantly remind you that you are working with a variable that may not have a value. There are several ways to access an optional. The first one we are going to discuss is **forced unwrapping**.

### Forced unwrapping

This is the easiest way to access Swift optional values. But it's also the most "dangerous". It is important to note that using an optional that has no value will cause an exception to be thrown in runtime. So forced unwrapping just instructs the program to get the value without any additional checks. You have to be absolutely certain that there is a value in the variable, otherwise you risk a crash. The following synthax:

	 myOptionalVariable!
	 
is how you unwrap your optional.

### Optional binding

Optional binding is the safest way to unwrap an optional. Consider the following code:

	if let nonOptional = myOptionalVariable {
		// use nonOptional safely
	}
	
Here a new variable (`nonOptional`) is created to hold the unwrapped value of `myOptionalVariable`. It is not an optional so it is safe to use it within the if statement's body. If `myOptionalVariable` has no value, the code inside the if statement will not be executed.

### Implicitly unwrapped optionals

Like many other features in the language, Swift optional values acknowledge common use cases and provide shorter ways to write them. Often, you will have an optional, but right from the start you will be fairly certain that it DOES in fact contain a value. So instead of using forced unwrapping everytime you use them, you can define those variables as implicitly unwrapped optionals like so:

	var myImplicitOptional: String!
	
This new variable is an optional in every way, but it differs in the way it's accessed. You don't have to forcefully unwrap it everytime you use it. Just write it's name without the exclamation mark. It's still going to raise an exception if there happens to be no value in it, but it spares you the hassle of unwrapping every time (developers are lazy).

### Optional chaining

The Swift programming language defines optional chaining as:

> “Optional chaining is a process for querying and calling properties, methods, and subscripts on an optional that might currently be nil”

Intuitively, optional chaining allows you to call methods and access properties that don't necessarily have a value. Like you would expect from Objective-C, if an optional in a chain has no value, the whole expression evaluates to nil.

To back these explainations with an example, optional chaining looks something like this:

	nonOptional.optionalProperty?.someMethod()
	
Note that you can achieve the same call (sort of) by simply typing:

	nonOptional.optionalProperty!.someMethod()
	
The problem, however, is that if `optionalProperty` happens to be `nil`, an exception will be thrown. Optional chaining on the other hand fails gracefully by not calling `someMethod` and returning nil as a result. In fact, all subsequent method calls winthin the same statement will not be called.

As you already saw from the example, optional chaining's synthax differs by replacing the exclamation mark with a question mark. That's what instructs Swift to use chainging instead of forced unwrapping.

# Conclusion

In conclusion, I'd like to say that Swift optional values are a must have for all iOS developers trying to switch from Objective-C. They are literary everywhere in the API. There is hardly a single class that does not have them. Also, they can take a little time to get used to. That's why it is important to pay them the attention they deserve.

Thanks for reading! 

[Swift vs Objective-C index]: http://devmonologue.com/ios/swift/swift-vs-objective-c/