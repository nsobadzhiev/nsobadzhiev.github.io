---
id: 303
title: Swift arrays
date: 2014-11-05T19:53:32+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=303
permalink: /swift/swift-arrays/
categories:
  - Swift
  - Swift vs Objective-C
tags:
  - array
  - Cocoa
  - collection
  - dictionary
  - ios
  - NSArray
  - NSDictionary
  - Swift
---
_This post is part of a [Swift vs Objective-C series][Swift vs Objective-C index] where we look at new features in Swift and how they compare to their predecesors from Objective-C.
In this article, we are going to discuss Swift arrays and dictionaries._

_Make sure to also visit the article about [Swift dictionaries][Swift dictionaries article]._

# Whats new with Swift arrays and dictionaries?

I kind of liked NSArray and NSDictionary, to be honest. I thought that they are easy to use and powerful. That's why I was a little disappointed to learn that Swift changed them so dramatically. Admittedly, collections in Objective-C are more suited to the "power user" - there are really amazing things they can do. But that comes at a price. It can be really easy to lose track of how you're using them and end up with messy code.

Apple's engineers re-envisioned Swift arrays by introducing a lot more restrictions and not allowing developers to go crazy with them.

## Swift arrays and dicitonaries eliminate primitive obsession

One of the major errors Swift arrays and dictionaries prevent, is [Primitive Obsession][primitive obsession definition]. Primitive obsession refers to a [code smell][code smell definition] that involves using primitive data types when you should really have a custom class that holds your data. Many times, in Objective-C this means storing a bundle of related but not homogeneous pieces of information in an array or dictionary. For instance, if you need to store a collection of username and password pairs, you might be tempted to allocate two arrays, one containing usernames and the other passwords, or maybe having an array of dictionaries, each containing a username and password keys. This might be the first thing that comes to mind, but it's really not a good idea. The right choice would be to create an Account class that contains properties for the information your application requires.

Since Swift arrays can only contain data with the same type (only strings or only numbers, for instance), developers have no choice but to use collections in a way that is easy to understand and maintain. 

That being said, I have to say that Swift arrays are a leap forward for the iOS development community. But I'll probably miss NSArray.

## Swift arrays contain objects of the same type

Swift does not allow you to add different types of data in the same collection. When you create it, you have to specify what kind of data you intend to store. Of course, the compiler uses type inference to guess the data type if you omit it. From then on, if you try to add an object from another class to your array, a compile time error will be raised. 

## Swift arrays mutability

In Objective-C, NSArray is immutable. If you needed to change a collection at runtime, you needed to use it's counterpart - NSMutableArray. In Swift, there is a more general solution to this problem. And it also works for strings, dictionaries and other primitive data. Mutability is controlled by the variable itself - a `var` is mutable, whereas a `let` is immutable.

	let myArray:Array<String> = ["Swift", "Array", "Dictionary"]
	myArray.append("Collection")
	
This code generates the following error - "Immutable value of type 'Array<String>' only has mutating members called 'append'". Descriptive as always...
However, if we declare `myArray` as a variable and not a constant, the code will work:

	var myArray:Array<String> = ["Swift", "Array", "Dictionary"]
	myArray.append("Collection")
	
After that statement, `myArray` contains `["Swift", "Array", "Dictionary", "Collection"]`

## The synthax of Swift arrays

### Creating Swift arrays

	let myArray:Array<String> = ["Swift", "Array", "Dictionary"]
	let myArray = ["Swift", "Array", "Dictionary"]	// type inference allows you to omit the type of the variable 
	
As you can see, Swift uses the same synthax for array literals that Objective-C introduced in it's most recent versions.

### Accessing and modifying array values

There are many ways to do that and most, if not all, you might remember from Objective-C

	let firstElement = myArray[0]
	let myArray = ["Swift", "Array", "Dictionary"]	// ["Swift", "Array", "Dictionary"]
	myArray.append("Last item")	// ["Swift", "Array", "Dictionary", "Last item"]
	myArray += ["Last item 2"]	// ["Swift", "Array", "Dictionary", "Last item", "Last item 2"]
	myArray.removeLast()		// ["Swift", "Array", "Dictionary", "Last item"]
	myArray.removeAtIndex(0)	// ["Array", "Dictionary", "Last item"]
	myArray[0] = "First"		// ["First", "Dictionary", "Last item"]
	
**Note:** Accessing or modifying an element beyond the array's bounds will raise an exception:

	myArray[13] = "13"    // raises an exception because 13 is beyond the arrays bounds
	let element13 = myArray[13] // also raises an exception
	
### Iterating arrays

Iterating Swift arrays is almost the same as in Objective-C:

	for string in myArray {
		print(string)
	} 

This code will print out every member of the array.

[Swift vs Objective-C index]: http://devmonologue.com/ios/swift/swift-vs-objective-c/
[primitive obsession definition]: http://sourcemaking.com/refactoring/primitive-obsession
[code smell definition]: https://en.wikipedia.org/wiki/Code_smell
[Swift dictionaries article]: http://devmonologue.com/ios/swift/swift-dictionaries