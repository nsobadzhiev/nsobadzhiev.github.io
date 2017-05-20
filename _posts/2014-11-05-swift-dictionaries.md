---
id: 312
title: Swift dictionaries
date: 2014-11-05T19:57:18+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=312
permalink: /swift/swift-dictionaries/
categories:
  - Swift
---
_This post is part of a [Swift vs Objective-C series][Swift vs Objective-C index] where we look at new features in Swift and how they compare to their predecesors from Objective-C.
In this article, we are going to discuss Swift dictionaries._

_Make sure to also visit the article about [Swift arrays][Swift arrays article]._

# Whats new with Swift dictionaries and arrays?

I kind of liked NSArray and NSDictionary, to be honest. I thought that they are easy to use and powerful. That's why I was a little disappointed to learn that Swift changed them so dramatically. Admittedly, collections in Objective-C are more suited to the "power user" - there are really amazing things they can do. But that comes at a price. It can be really easy to lose track of how you're using them and end up with messy code.

Apple's engineers re-envisioned Swift dictionaries by introducing a lot more restrictions and not allowing developers to go crazy with them.

## Swift dictionaries and arrays eliminate primitive obsession

One of the major errors Swift dictionaries (and arrays) prevent, is [Primitive Obsession][primitive obsession definition]. Primitive obsession refers to a [code smell][code smell definition] that involves using primitive data types when you should really have a custom class that holds your data. Many times, in Objective-C this means storing a bundle of related but not homogeneous pieces of information in an array or dictionary. For instance, if you need to store a collection of username and password pairs, you might be tempted to allocate two arrays, one containing usernames and the other passwords, or maybe having an array of dictionaries, each containing a username and password keys. This might be the first thing that comes to mind, but it's really not a good idea. The right choice would be to create an Account class that contains properties for the information your application requires.

Since Swift dictionaries can only contain data with the same type (only strings or only numbers, for instance), developers have no choice but to use collections in a way that is easy to understand and maintain.

That being said, I have to say that Swift dictionaries are a leap forward for the iOS development community. But I'll probably miss NSDictionary.

## Swift dictionaries contain objects of the same type

Swift does not allow you to add different types of data in the same collection. When you create it, you have to specify what kind of data you intend to store. Of course, the compiler uses type inference to guess the data type if you omit it. From then on, if you try to add an object from another class to your dictionary, a compile time error will be raised.

## Swift dictionaries mutability

In Objective-C, NSDictionary is immutable. If you needed to change a collection at runtime, you needed to use it's counterpart - NSMutableDictionary. With Swift dictionaries, there is a more general solution to this problem. And it also works for strings, arrays and other primitive data. Mutability is controlled by the variable itself - a `var` is mutable, whereas a `let` is immutable.

let myDict:Dictionary&lt;String, String&gt; = ["Swift": "language", "Array" : "Collection", "Dictionary": "Collection"]
myDict.removeValueForKey("Swift")

This code generates the following error - "Immutable value of type 'Dictionary&lt;String, String&gt;' only has mutating members called 'removeValueForKey'". Descriptive as always...
However, if we declare `myDict` as a variable and not a constant, the code will work:

var myDict:Dictionary&lt;String, String&gt; = ["Swift": "language", "Array" : "Collection", "Dictionary": "Collection"]
myDict.removeValueForKey("Swift")

After that statement, `myDict` contains `["Array" : "Collection", "Dictionary": "Collection"]`

## The synthax of Swift dictionaries

### Creating a dictionary

let myDict:Dictionary&lt;Int, String&gt; = [1: "One", 2: "Two"]
let myDictShort:[Int: String] = [1: "One", 2: "Two"]
let myDictShorter = [1: "One", 2: "Two"]

The first statement is the longest definition of a dictionary. The second one is a shorthand version of the first, and it's prefered. The third definition just uses type inference to omit the data type.

### Accessing members

Accessing dictionary members is almost the same as accessing array elements. In fact, an array might be thought of like a dictionary where the keys are the numbers from 0 to N, where N is the size of the array.

var myDict2 = ["One": 1, "Two": 2]
let element = myDict2["One"]
let noElement = myDict2["false Key"] // returns nil - no such element
myDict2["Three"] = 3
myDict2.updateValue(4, forKey: "Three")
myDict2.removeValueForKey("Three")

### Iterating dictionaries

In Objective-C, iterating over all dictionary entries was a little more difficult than it should have been. In Swift, it is a lot easier due to tuples:

for (number, numberString) in myDict2 {
print(number)
println(numberString)
}

Each time execution goes in the loop's body, the tuple is filled with a different pair taken from the dictionary. The `(number, numberString)` statement "parses" the tuple into the `number` and `numberString` variables so they can be used inside the loop. It's short trick that allows you to write simple for loops.

[Swift vs Objective-C index]: http://devmonologue.com/ios/swift/swift-vs-objective-c/
[primitive obsession definition]: http://sourcemaking.com/refactoring/primitive-obsession
[code smell definition]: https://en.wikipedia.org/wiki/Code_smell
[Swift arrays article]: http://devmonologue.com/ios/swift/swift-arrays