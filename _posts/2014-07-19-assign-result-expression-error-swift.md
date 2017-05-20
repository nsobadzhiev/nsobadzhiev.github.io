---
id: 257
title: Cannot assign to the result of this expression error in Swift
date: 2014-07-19T18:38:04+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=257
permalink: /swift/assign-result-expression-error-swift/
categories:
  - Swift
tags:
  - Cannot assign to the result
  - compile error
  - optional value
  - Swift
---
# How to fix the Cannot assign to the result of this expression error in Swift

I few days ago I encountered the following error in Xcode:

![Cannot Assign Result Error]({{ site.url }}/images/2014/07/cannot_assign_result_error.png "An Xcode screenshot showing the Cannot assign to the result of this expression error in Swift for the statement - self.urlDownloader?.delegate = self")

The problem was with the following statement:

	self.urlDownloader?.delegate = self

As always, Swift's compiler continues Objective-C's tradition of showing criptic error messages that are hard to understand. I spent far more time than I should have on this one so I thought I should write a post about it so that you don't have to. 

The fix turned out to be pretty simple. You just have to replace the question mark with exclamation point like so:

	 self.urlDownloader!.delegate = self
	 
# But why?

The "Cannot assign to the result of this expression" error message can be a bit obscure and does not give any pointers as to why this is a problem. I think that since we use optional chaining (by writing the question mark), it is possible for the left hand side of the expression to evaluate to `nil`.

_**Note:** Optional chaining means that if the `delegate` property turns out to be `nil`, the whole expression will not continue evaluating and `nil` will be return as a result. You can read more about [optional values here][Post about optional values]._

So, if the expression on the left results to `nil`, the statement resolves to:

	nil = self
	
which is clearly an error. By replacing the question mark with an exclamation point, the delegate property is _force unwrapped_, which makes sure the expression is evaluated to a real variable. That's why it fixes the Cannot assign to the result of this expression compilation error.

[Post about optional values]: http://devmonologue.com/ios/uncategorized/swift-optional-values/