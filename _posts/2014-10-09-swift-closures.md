---
id: 291
title: Swift Closures
date: 2014-10-09T20:55:07+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=291
permalink: /ios/swift-closures/
categories:
  - iOS
  - Swift
  - Swift vs Objective-C
  - Tutorials
tags:
  - blocks
  - closure
  - ios
  - Swift
---
_This post is part of a Swift vs Objective-C series where we look at new features in Swift and how they compare to their predecesors from Objective-C. In this article, we are going to discuss Swift closures._

# What are Swift closures?

Depends who's asking...
To an **Objective-C developer**, a closure is Swift's equivilent of blocks.
A **formal definition** should be that Swift closures are _self-contained blocks of functionality_
**Intuitively**, a closure is a kind of anonymous, nested functions. It's a chunk of functionality that you can pass around in functions and variables.

In reality, closures and blocks do little more than provide a more convinient way of encapsulating functionality than nested fuctions and function pointers. Despite that, in the past couple of years, there has been a massive effort to integrate blocks into our Objective-C code.
 
# Swift closures synthax

Swift closures closely resemble Objective-C (or any other language, supporting them) blocks. To define one, you'd write something like this:

	{ (parameters) -> return_type in
    	// block body
		// write code here
	}

This is longest, most formal way of creating Swift closures. As with all languages influenced by functional programming, there are many shorter versions, which we will cover shortly.

## Closures as function parameters

Let's face it, you are most likely to use Swift closures as parameters. Let's take a look at some specifics to doing that.
First of all, it is important to know how to define a function that takes swift closures as one of it's parameters:

	func calculateSomething(something1:Float, something2:Float, calculator:(value1:Float, value2:Float) ->Float) ->Float {
    	return calculator(value1: something1, value2: something2)
	}
	
I hope there is nothing disturbing in this statement to you. It's a function that takes three parameters - two floating point numbers and another one called calculator. When we inspect it's type, we will discover that it's not a basic data type or a class. It's a function signature, specifying what parameters the closure takes and what value it returns.

Now, to use the `calculateSomething` function, we can do the following:

	calculateSomething(23.76, 125, {(value1:Float, value2:Float)->Float in 
    	return value1 + value2
	})
	
Written like this, the function is calculating sums. As you can see, we supplied two numbers and a closure that is essentially a very complicated `+` operator. However, as with most "functional" languages, that is simply too long to write. Life is too short to write swift closures with the full synthax. That's why there are shorter versions of this statement.

	calculateSomething(23.76, 125, {(value1:Float, value2:Float) in 
    	return value1 + value2
	})
	
Type inference to the rescue! Since the compiler already knows the signature needed for this function call, it can "guess" what a suitable closure would look like. That's why we can skip explicitly specifying it's return type. You might wonder if the same is true about the parameter's type. Well, it is... 

	calculateSomething(23.76, 125, {(value1, value2) in 
    	return value1 + value2
	})
	
All data types can be inferred from the function definition.
While we are at it, we can also omit the parenthesis:

	calculateSomething(23.76, 125, {value1, value2 in 
    	return value1 + value2
	})

Next, we can continue with the closure's body. In swift, you can omit the `return` keyword for functions that have only one statement. Since this is the case in our example, we can shorten our code even more:

	calculateSomething(23.76, 125, {value1, value2 in 
    	value1 + value2
	})

But wait, there's more! Here's an interesting little feature from scripting languages. You might know (from Bash for instance) that you can access function and script paramenters using the `$` synthax (like `$0`, `$5`, `$8`). So...
	
	calculateSomething(23.76, 125, {
    	$0 + $1
	})
	
Note that inlike Bash, where `$0` is the path to the script being executed, in Swift it is actually the first parameter.

Pretty nice, eh? Our function call got a lot shorter. Of cource, this is just [synthax sugar][Synthax sugar wiki page]. It doesn't add any expressive power to swift closures - it just makes them more convenient to write and read.

As a bonus for being so patient, here's one final short form. Since the closure we were using sums two numbers, which is exactly what `+` does... we might as well use that operator:

	calculateSomething(5.0, 12.6, +)
	
Yes, that's a thing in Swift.

## Variables of type closure

Sometimes, it can prove useful to store swift closures in a variable. So, let's briefly talk about that. 

	let closureVar = {(value1:Float, value2:Float)->Float in 
    	return value1 + value2
	}

By now, you should be comfortable with this synthax. It's not much different than what we looked at above. Though, it looks like a long this to write. It almost makes you not want to use that variable. Can't we use something shorter, like we did last time?

	let closureVar2 = {(value1, value2) in 
		return value1 + value2
	}
	
It doesn't work ;(. Why? Well, we can omit the parameter data types and return value only if the compiler can infer them from the context. And since we didn't specify the `closureVar2`'s type, Swift doesn't know the closure's signature that it should expect.
	
	let closureVar3:(value1:Float, value2:Float) ->Float = {(value1, value2) in 
    	return value1 + value2
	}

Explicitly adding a type to our variable solves that problem, but our statement is no longer that short.

Now that we have a variable containing our closure, we can use it in all situations a closure is needed. Like in all examples from the previous paragraph:

calculateSomething(5.0, 12.6, closureVar)
 
# Capturing values

_This section contains some general information about capturing values. If you are familiar this concept from Objective-C's blocks, feel free to skip the following paragraph._

Everything up until now is not that impressive to be honest. Being able to store functions in a variable and use them as parameters is a standard feature of most languages. What makes Swift closures (also blocks and lambda functions in Objective-C and other languages) so popular, is capturing values. The concept is rather simple. Unlike conventional function that operate in a sort of a vaccum, closures have the ability to use variables defined in the code block enclosing them. Let look at an example:

	func myFunction() {
    	var funcVariable = 15
    	let closureVar: () = {() -> () in
        	// This closure will be able to access variables 
        	// from the function it was defined from -
        	// funcVariable for instance
        	var closureVariable = funcVariable
        	funcVariable = 25
    		}()
    	closureVar
    	print(funcVariable)
   		// funcVariable is now 25
	}
	
The outer function `myFunction` creates a local variable `funcVariable`. After that it defines a closure, which is able to use that local variable and even change it. Actually it has a reference to it so that if the value is modified within the closure, the new value doesn't get lost as soon as the code block ends.

# Retain cycles with Swift closures

Capturing values is a pretty useful feature. However, there are some things to be careful with. As the title suggests, they are related to retain cycles. Since the closure has to make sure all local variables that it needs from it's container are readily available within it's body, it retains them. It is possible that an object retains a closure, and that same object is used within the closure's body, meaning that they both keep each other "alive". Most often this happens when a class has a closure as a member variable and that same closure uses `self` within it's body. In such cases, the object retains the closure, which in turn retains the object of the class. 

Retain cycles like these are relatively hard to track because they are very subtle. And with ARC, things are getting even worse as most developers lose the habbit of thinking about memory management.

[Synthax sugar wiki page]: http://en.wikipedia.org/wiki/Syntactic_sugar