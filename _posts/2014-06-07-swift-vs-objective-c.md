---
id: 217
title: Swift vs Objective-C
date: 2014-06-07T16:32:23+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=217
permalink: /swift/swift-vs-objective-c/
categories:
  - Swift
  - Swift vs Objective-C
tags:
  - ios
  - ObjC
  - Objective-C
  - Swift
  - Swift vs Objective-C
---
![Swift vs Objective c]({{ site.url }}/images/2014/06/swift_vs_objective_c.png "Picture containing the Swift and Objective-C logo with vs written between")
# A new programming language

In case you've been living under a rock, Swift is the new programming language, developed by Apple. It is designed to completely replace Objective-C for Mac OS and iOS developement. They totally blew out minds this year. One day we were seasoned iOS developers with years of experience and the other... we are still struggling with the language synthax. I'm exaggerating ofcourse. Programming is much more than the language we write, or the API that we use. And if you think that Swift is going to severely impact your capabilities as a developer, you should rethink your mentality as a software developer. Being a programmer is all about the journey, a never ending cycle of keeping up with the latest technologies. That's why it's so hard if you don't love what you're doing - you don't have the constant urge to learn more. iOS itself is a pretty new platform to begin with. And the same way you chose to adopt it despite that, you should also give Swift a chance.

Wow... that was inspirational... like every other motivational speech you ever heard. But it's true. And don't be offended - I just want to motivate you to learn Swift.

# Swift &gt; Objective-C???

But why did Apple decide to replace Objective-C - it was just becoming more and more awesome?
It is my observation that iOS and Mac developers love the language. Apart from newbies to the platform that complain about the synthax (mainly the square brackets), everyone seems to agree that Objective-C is rather intuitive and easy to work with. Additionally, in the last couple of years, with the addition of features like ARC (Automatic Reference Counting), Storyboards and Autolayout, writing code for Apple product has, no doubt, become considerably easier and more flexible. And yet... it got replaced.

> Sorry, Objective-C, you're a great person, but I just don't feel the same about you... I met someone else...

I'm honestly not sure why Apple chose to use a different programming language. I suspect that they thought that having a language that's compatible with C and C++ meant that you had to use some _low level_ patterns such as pointers, references and memory management (well, not with ARC). In my opinion such features are so simplified in Objective-C (when was the last time you **really** had a problem with pointers) that it seems as if they are not there. Everything is handled at a higher level of abstraction. To me, Java and Objective-C have much more in common in the way thay make you think than Objective-C and C/C++.

That being said, I understand why a newcomer to the platform would feel intimidated by these things. And that's why it makes sense to remove them completely. And you cannot just have Objetive-C without pointers and C... You need something new. You need Swift.

# Swift vs Objective-C

Right after I heard that Apple has developed Swift and as I started reading about it, I immediately started thinking about how they are trying to fix common issues that developers had with Objective-C. That's precisely what this article is all about - Objective-C issues fixed in Swift.

An obvious answer is that it provides a higher level of abstraction. Swift is really similar to scripting languages such as Python and Ruby. It leaves the same functional programming vibe. Lambda functions, implicit data types, functions such as map, filter reduce etc. Objective-C doesn't have that (although frameworks such as [ReactiveCocoa][ReactiveCocoa home page] can achieve that - you can read about it [in this tutorial][ReactiveCocoa article]) - it's all procedural. It's true that such languages allow you to express your thoughts in code easier. Inlike C and C++ you don't get distracted with memory management and data structures and you can focus on what you are _actually_ trying to achieve.

A key advantage Swift has over scripting languages is that it is actually compiled. It resolves to a binary that can by executed by the CPU, rather than interpreted by a virtual machine. That makes it faster. Apple says that Swift is even faster that Objective-C itself, though I don't expect to see any sizable difference.

However, the focus of this article is not these kinds of differences. Setting performance and low-level vs high-level languages aside, I want to talk about things like language features and things that were just annoying in Objective C, but Swift made better.

Since this subject can get too long for a single post, I'll try to cover each topic in a separate article. To give you an idea of what I plan to write about, I've compiled a list of all the things that come to mind right now:

* (Swift vs Objective-C) Array and Dictionary vs. NSArray and NSDictionary

 There are a lot of differences between the way Objective-C and Swift handle collections. To the untrained eye, it might seem that iOS developers loose some flexibility by using Swift, but this is not necessarilly so.

* (Swift vs Objective-C) [Optional values]

 Optional values are an interesiting feature that is not too common in programming languages nowadays. They are a bit odd, but definitely worth your time. They replace the way Objective-C handle method calls on nil objects

* (Swift vs Objective-C) [Designated and convenience initializers]

 Swift continues Objective-C's tradition of distringuishing contructors, but also introduces some synthax that will prevent confusion between the two.

* (Swift vs Objective-C) [Closures (blocks in Swift)]

 Swift's equivilent to blocks are closures. They provide the same functionality, but give some more flexibility with memory management

* (Swift vs Objective-C) Swift external and internal parameter names

 A key readability feature in Objective-C is that method names can provide hints to developers for each parameter that they take. This tradition is continued in Swift by providing external and internal names.

* (Swift vs Objective-C) [Enumerations in Swift]

 Swift's take on enums provides so much more flexibility that you'll wish someone thought about it years ago

* (Swift vs Objective-C) Swift basic data types

 In Swift, basic data types such as integers and characters can be used alongside object references seamlessly, unlike in Objective-C, where you had to treat NSString and NSUInteger (for instance) separately.

* (Swift vs Objective-C) Control flow in Swift

 Even the control flow operator such as for, while and if are improved. They re much more similar to they ones in scripting languages. 

[ReactiveCocoa home page]: https://github.com/ReactiveCocoa/ReactiveCocoa
[ReactiveCocoa article]: http://devmonologue.com/ios/frameworks/reactivecocoa/
[Optional values]: http://devmonologue.com/ios/uncategorized/swift-optional-values/
[Designated and convenience initializers]: http://devmonologue.com/ios/swift/swift-initializers-designated-convenience/
[Enumerations in Swift]: http://devmonologue.com/ios/swift/enumerations-swift/
[Closures (blocks in Swift)]: http://devmonologue.com/ios/ios/swift-closures/