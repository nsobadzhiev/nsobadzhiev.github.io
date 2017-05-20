---
id: 276
title: Using Objective-C in Swift
date: 2014-09-02T20:24:36+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=276
permalink: /swift/using-objective-c-swift/
categories:
  - Swift
tags:
  - bridging header
  - ios
  - Objective-C
  - objective-c in swift
  - Swift
---
As we start introducing Swift in our projects, we will, no doubt, end up in a situation where part of our code is in Objective-C and the other is in Swift. So we should find a way to make them "get along" with each other. In the Swift programming language book, Apple promises that Swift can work seemlessly with Objective-C, however they do little to illustrate how that works. In this article, we are going to focus on just that - how to import and use your "legacy" Objective-C code in your newly written Swift files.

# Importing Objective-C classes in Swift code

We all know Apple stated that importing Objective-C classes works out-of-the-box in Swift, but as soon as you attempt to actually do it, you realize something - you cannot include a class's header file the same way you could before. So how do you tell the compiler you want that particular class to be imported into your Swift code? The answer to that question is something called a **bridging header**.

## Bridging headers

Bridging headers are the backbone that connects your Objective-C and Swift code. The first time you add a Swift file to an Objective-C project or an Objective-C file to a Swift project, Xcode will prompt you to generate a bridging header.

![Bridge Header Prompt]({{ site.url }}/images/2014/09/bridge_header_prompt.png "Xcode asks you to generate a bridging header")

Ofcourse, you can generate the Objective-C bridge file yourself by creating a new header file and making sure it's path is correctly indicated under Build Settings -&gt; Swift Compiler - Code Generation.

Now lets talk about the bridge file's contents. The Objective-C bridging header contains a series of import statements that identify every piece of code you want exposed to Swift. If you want to use a class you previously wrote in Objective-C, you just add it to the header. When you do that, all functionality inside of it is available to all swift files in your project. 

_You've probably realized by now that Swift doesn't really rely on import statements in order to use code from other files. When you add a Swift file to a project, it is implicitly available whenever you need it. Apart from importing frameworks and modules, you are rearly expected to worry about that._

This is what you might expect the header to look like:

	//
	//  Use this file to import your target's public headers that you would like to expose to Swift.
	//

	#import "AnObjCClass.h"
	#import "AnotherObjCFile.h"
	
That's all you need to do.

## Using Objective-C in Swift - converting Objective-C statements

There's one more detail we need to iron out in order to finish talking about using objective-C in Swift. The problem is that you cannot just start writing Obj-C statements in your Swift files. It's doesn't work that way. However, the solutions is easier than you might have thought.

Since Apple announced Swift, leaving us, the developers, having as much experience with the SDK as a novice programmer, there was a single piece of information that still gave us advantage. We knew the API. The language might have changed comppletely, but classes and methods retained their interfaces (mostly). We didn't have to scroll through the documentation to find the functionality we needed.

What's more, Apple didn't achieve that by rewriting all APIs. They used the existing Objetive-C interfaces and converted them to Swift. And that exactly how using Objective-C in Swift works. You don't use the old synthax, but rather the interface is automatically ported to Swift so that you can access your functionality via the new language's statements.
For example, an Objective-C statement such as:

	[connection startDownloding:request];
	
becomes:

	connection.startDownloading(request)
	
and:
	
	connection = [[Connection alloc] initWithUrl:url];
	
is converted to:
	
	connection = Connection(url:url)
	
That way your Swift files don't get cluttered with Objective-C code, unlike what was happening previously with C++. 

So that's how using Objective-C in Swift works. For more information regarding bridging headers, please refer to [Swift and Objective-C in the Same Project][Apple docs on bridging headers]

[Apple docs on bridging headers]: https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html