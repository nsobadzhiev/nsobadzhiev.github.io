---
id: 158
title: Writing Objective-C unit tests
date: 2014-03-22T20:27:56+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=158
permalink: /test-driven-development/writing-objective-c-unit-tests/
categories:
  - Test Driven Development
tags:
  - ios
  - OCUnit
  - TDD
  - Test Driven Development
  - Xcode
---
<i>This post is the third part of the Test Driven Development series, guiding you in the process of integrating test driven development in your iOS projects. Previously, in parts <a title="Introduction to Test Driven iOS development" href="http://devmonologue.com/ios/test-driven-development/introduction-to-test-driven-ios-development/">one</a> and <a title="Test Driven Development with OCUnit" href="http://devmonologue.com/ios/test-driven-development/test-driven-development-ocunit/">two</a>, we covered the mechanics of setting up Xcode with OCUnit, as well as some general concepts of TDD. So now it is time to dig a little bit deeper and discuss how to write meaningful tests.</i>

<span style="line-height: 1.5em;">If you've read the previous articles in the series, you should be able to setup Xcode and OCUnit for test driven development. However, if you have tried it, you might have felt a bit lost when it comes to writing an actual unit test. In my experience, TDD requires some practice before you are able to do it without struggling. And that's what this article is about – it will give you some pointers on how to go about writing Objective-C unit tests – how to approach the problem and what techniques to use.</span>
<h3>How to write Objective-C unit tests</h3>
<span style="text-decoration: underline;"><b>Test a single piece of functionality</b></span>

It is <span style="text-decoration: underline;">very</span> important that your unit tests check for only one thing. Don't be tempted to verify several properties at once. For instance, if you are writing a calculator application and want to test your multiplication algorithm, your most important unit test will be the one that proves that you are able to multiply two numbers. But you might also want to try going beyond the bound of the integer data type, or using negative numbers. However, you shouldn't do that at the same time. These are three distinct pieces of functionality and should be handled separately. Write one test for multiplying simple numbers like 2 and 4 so that you know your algorithm is working in the most straight-forward case and after that you can write another one using negative values, zero and big numbers that go outside the bounds of integer.
<p align="LEFT">But why is this so important? First of all, it results in simpler and easier to read unit tests. Remember that one of the key advantages of TDD is that other developers can read your tests to quickly learn how your code is supposed to be used. Secondly, if you wrote only one unit test in the example above, and it happened to fail for some reason, you would have had a hard time discovering what caused the failure. You would know that the multiplication algorithm failed, but was it the part that handles negative numbers, the checks for going beyond the integer max value or the core functionality? You wouldn't have had that problem if you had three separate tests.</p>
<p align="LEFT"><span style="text-decoration: underline;"><b>Write appropriate unit test names</b></span></p>
<p align="LEFT">You should take the time to write descriptive names for your test methods. This is helpful when one of them fails – Xcode will print it's name and if it hints the functionality that is incorrect, it is going to help you find the problem quickly. And also, it makes the code easier to understand.</p>
<p align="LEFT">The name should describe the exact scenario executed in the test so that other developers don't have to read the method's body to understand what it does – something like “testCalulatorMultipliesNegativeNumberWithZero”. It will not be very pleasant to look at but it can help a lot.</p>
<p align="LEFT"><span style="text-decoration: underline;"><b>Write tests from the bottom up</b></span></p>
<p align="LEFT">Another great tip I can give you is related to the way you write your actual test method. Once you decide what functionality you are going to verify and have created an empty method, instead of writing your code from the beginning towards the end, try doing the opposite. A unit test has three distinct parts. The first one sets up the environment for the test – it initializes objects, sets values and overall prepares the program for executing the code under test. The second part is the actual invocation to the tested functionality and the third part verifies that the program worked correctly.</p>
<p align="LEFT">So, instead of writing these three steps in sequence, try implementing the error checking one first. Once that's done, add the first two parts accordingly.</p>
<p align="LEFT">The idea here is that if you think of a sample value to test with, it isn't always easy to find the right solution for it (after all, that's why you are writing an application to do it for you). The reverse, on the other hand, is often a lot easier. Pick a solution and figure out what problem it solves.</p>
<p align="LEFT">An example that illustrates this concept is finding a string in text. Looking for all occurrences can be difficult and error-prone thing to do by hand. But if you start with the final result (let's way three occurrences at indexes 23, 54 and 346) it will be a lot easier to add random data between these indexes to come up with the initial text.</p>
<p align="LEFT"><span style="text-decoration: underline;"><b>Use mock objects</b></span></p>
<p align="LEFT">Mock objects are instances of classes that you made specifically to help in testing. Let's say you have to test some code that depends on the file system to get it's job done. Since you shouldn't rely on the actual file system for your unit test, you need to replace that functionality with your own. Essentially, you need to override all methods that interact with the “real” file system and replace them with hardcoded values. So, when the class, being tested asks for the contents of the file at a given path, instead of actually reading it, your overridden methods return a hardcoded piece of data.</p>
<p align="LEFT">There are two ways you can achieve this. The easiest is to identify all methods that read file system data and create a subclass that overrides them and returns test data. However, there might be many such methods and they might have additional functionality (not only interacting with the file system). Overriding all of them is not always a good idea and ideally, you don't want to replace much of the production code to avoid something that works in your tests break in production because you have overridden some functionality.</p>
<p align="LEFT">The second (and better) way to override all file system interactions is to not have any in your production class in the first place. Instead of having the class under test interact with the file system directly, have a separate class that does it and let the tested class have an instance of it. Then, you can replace that new class with a mock implementation that doesn't actually read any files, but rather returns a predefined set of data. Such a scenario is depicted in the sample below:</p>

<pre class="brush:objc">/**

* unit test

*/

- (void)testBeingAbleToDoSomethingImportant

{

ClassUnderTest* testClass = [ClassUnderTest new];

XCTAssertNoThrow([testClass methodUnderTest], @"methodUnderTest shouldn't throw and exception");

}</pre>
<pre class="brush:objc">/**

* unit test with mock objects

*/

- (void)testBeingAbleToDoSomethingImportant

{

MockClassUnderTest* testClass = [ClassUnderTest new];

testClass.fileInteractionManager = [MockFileManager new];

XCTAssertNoThrow([testClass methodUnderTest], @"methodUnderTest shouldn't throw and exception");

}</pre>
<pre class="brush:objc">/**

* Production classes

*/

@inteface ClassUnderTest : NSObject

{

FileManager* fileManager;

}

- (void)methodUnderTest;

@end

@implementation ClassUnderTest

- (void)methodUnderTest

{

NSData* someFileData = [fileManager readSomeFileFromPath:@"/test/path"];

// do something with the data

}

@end

@interface FileManager : NSObject

- (NSData*)readSomeFileFromPath:(NSString*)path;

@end

@implementation FileManager

- (NSData*)readSomeFileFromPath:(NSString*)path

{

return [NSData dataWithContentOfFile:path];

}</pre>
<pre class="brush:objc">/**

* Mock classes

*/

@inteface MockClassUnderTest : ClassUnderTest

@property(nonatomic, strong) FileManager* fileInteractionManager;

@end

@implementation MockClassUnderTest

- (void)setFileInteractionManager:(FileManager*)fileInteractionManager

{

fileManager = fileInteractionManager;

}

- (FileManager*)fileInteractionManager

{

return fileManager;

}

@end

@interface MockFileManager : FileManager

@end

@implementation MockFileManager

- (NSData*)readSomeFileFromPath:(NSString*)path

{

// return empty data for simplicity.

// A more useful implementation might have a property that

// can be used to set the NSData that will be returned from

// this method

return [NSData data];

}

@end</pre>
<p align="LEFT">In the code snippet above, two mock objects are created – MockFileManager and MockClassUnderTest. MockFileManager is used in order to override the “readSomeFileFromPath:” method and return a predefined value from it that does not depend on the real file system on the device. MockClassUnderTest is used in order to replace the standard instance of FileManager with the mock one. It just adds a property that can be used to set the “fileManager” member variable that is used by MockClassUnderTest. You might get away with adding that property in MockClassUnderTest itself, but it is generally not recommended because functionality shouldn't be added to a production class solely for the purpose of unit testing. If a property should be private for the production code, it needs to stay that way even though access to it is needed for testing.</p>
<p align="LEFT">The second solution might be more complex, but in reality it is the right way to do it. It is an example of how test driven development forces you to write better code. Since it is difficult to test code with a lot of dependancies, you instinctively write code with greater structural (and functional) quality.</p>
<p align="LEFT">So even though it is considerably easier to have all file system interaction code in the classes that need it, you take the time to extract it into another layer of abstraction because it will make it easier to unit test.</p>
<p align="LEFT"><span style="text-decoration: underline;"><b>Singleton vs. single instance</b></span></p>
<p align="LEFT">The singleton is a popular design pattern where a class is only allowed to have one instance. This so called “shared instance” is used throughout the application and every other class has access to it. On the other hand, a single instance refers to a class that is not a singleton (and does not actively enforce that you cannot create several instances of it) but is only instantiated once.</p>
<p align="LEFT">Even though a singleton is more convenient for developers in the sense that they don't have to worry about getting a reference to the object when they need it (they just call the “sharedInstance” class method and the instance is returned), it takes away a but of flexibility and can lead to some difficulties when using test driven development. Imagine that in the code example above, the file manager class was a singleton (which can be the case with NSFileManager if using the “defaultManager”). In the unit tests, we need to replace that object with a mock one. However, it is not easy because the class directly calls the class method that returns the singleton instance. In Cocoa, we can use the Objective-C runtime in order to change what that class method does and make it exchange the singleton instance with a mock one, but other languages don't have that. And this is hardly an elegant solution.</p>
<p align="LEFT">That's why it is recommended to choose single instance, instead of singleton. You can create an object of the class and pass it along to the classes that need it.</p>
<p align="LEFT">I think that's it for now. It is by no means a complete set of rules to follow when using test driven development but it is something that has helped me a lot. Make sure to come back for the next TDD article. Once again, thanks for reading.</p>