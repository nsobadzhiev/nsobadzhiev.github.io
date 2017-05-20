---
id: 110
title: Test Driven Development with OCUnit
date: 2014-02-16T11:52:12+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=110
permalink: /test-driven-development/test-driven-development-ocunit/
categories:
  - Test Driven Development
tags:
  - ios
  - ObjC
  - Objective-C
  - OCUnit
  - TDD
  - Test Driven Development
---
<p style="text-align: left;" align="CENTER"><b></b><i>This post is the second part of the Test Driven Development series. It will guide you in the process of integrating test driven development with OCUnit in your iOS projects. Last time, <a href="http://devmonologue.com/ios/2014/01/29/introduction-to-test-driven-ios-development/">in part one</a>, we covered some test driven development basics and now it's time to actually prepare a project for unit testing.</i></p>

<h2>Xcode and TDD</h2>
<p style="text-align: left;" align="CENTER"><b></b>It is a common misconception that test driven development is not suitable for mobile applications. Far from it! Xcode has unit testing integration since version 2.1. Do you remember Xcode 2.1? I certainly don't, because I hadn't started developing for iOS. Neither had most of you, I assume. So nowadays, TDD is just a few clicks away. And that's exactly today's topic.</p>
 There are several unit testing platforms available to iOS (and Mac) developers:
<ul>
	<li>OCUnit</li>
	<li>GHUnit</li>
	<li>CATCH</li>
	<li>...</li>
</ul>
<i><b>Disclaimer: </b>I've only used OCUnit :)</i>

Let me introduce them to you:
<h4>OCUnit:</h4>
OCUnit is the single, most popular TDD framework for iOS development. Why? Well... uhm... it's integrated in Xcode.
<h4>GHUnit:</h4>
GHUnit is another popular unit testing platform for iOS. I can only pretend to know anything about it other than that it compensates for not being integrated in Xcode by providing a user interface, where it shows all tests that are executed and the results from them. While both OCUnit and GHUnit require a new target to be created in Xcode solely for testing, GHUnit needs you to add it's own application files along with your testing classes and mock objects, making the setup process a little more complicated. If you are interested, you might consider visiting it's <a href="https://github.com/gh-unit/gh-unit">github</a> page.
<h4>CATCH:</h4>
CATCH is a framework that is written in C++ and hence, it can be easily used to test your C/C++ code. If that's important to you, visit it's <a href="https://github.com/philsquared/Catch">github</a> page.

In this article, we're going to focus on OCUnit because it's easiest to setup and works best with Xcode. Later on, I might consider writing a separate post about it's alternatives.

<i>A philosophical question one might ask is “Are a unit testing frameworks unit tested during development and what with?”. The answer I found was “Yes” and “With the same framework”. Please refer to <a href="https://github.com/gh-unit/gh-unit/tree/master/Tests">GHUnit's GHUnit tests</a> and <a href="https://github.com/philsquared/Catch/tree/master/projects/SelfTest">CATCH's CATCH tests</a>.</i>
<h2>Creating a new Xcode project with unit tests</h2>
It is now time to show you how to setup your new projects with test driven development in mind. More specifically, I was going to show you multiple screenshots of me essentially clicking an “Include Unit Tests” checkbox like it's rocket science. But as I went through Xcode's new project wizard, I realized that there is no such option. Apparently, you cannot opt to exclude unit testing from your new project in the latest version of Xcode. Either way, I'm going to add a few images because an article without pictures seems boring:

[caption id="attachment_118" align="aligncenter" width="605"]<a href="{{ site.url }}/images/2014/02/newProjectTemplate.png"><img class="size-large wp-image-118" alt="The image shows Xcode's new project template, where developers can choose a template for their new application" src="{{ site.url }}/images/2014/02/newProjectTemplate-1024x690.png" width="605" height="407" /></a> New Project Wizard - choosing application template[/caption]

Every tutorial I've ever read starts with the single view template. So I'm going to do something radical and use the empty application. Next, in Figure 2, I choose the name for my new project – Dev Monologue – an application that allows my readers to enjoy the blog on the go (though I wouldn't get too excited that it's going to hit the AppStore any time soon). Depending on the version of Xcode you use, you might see an “Include Unit Tests” checkbox. In the latest update, it is missing, but I remember that previously it was there. So in reality, those of you who update their macs, should be able to get a TDD-ready project without any extra effort.

&nbsp;

[caption id="attachment_117" align="aligncenter" width="605"]<a href="{{ site.url }}/images/2014/02/newProject.png"><img class="size-large wp-image-117" alt="The image shown the section of Xcode's new project wizard where the project name is chosen and some application-wide parameters are set - Core Data usage and Unit testing " src="{{ site.url }}/images/2014/02/newProject-1024x690.png" width="605" height="407" /></a> New Project Wizard - choosing project name[/caption]

After all that, we should be presented with our newly created project. It is almost empty right now, but there are still some goodies, generated for us by Xcode.

[caption id="attachment_114" align="aligncenter" width="605"]<a href="{{ site.url }}/images/2014/02/tddProject.png"><img class="size-large wp-image-114" alt="The image shown the newly created project opened in Xcode" src="{{ site.url }}/images/2014/02/tddProject-1024x621.png" width="605" height="366" /></a> The new project opened in Xcode[/caption]

Alongside the usual AppDelegate, Info.plist and project file, you will also notice that in the project navigator there is a group called “Dev MonogolueTests”. That's where our unit tests and mock objects go, but more on that later on. Also, there is a second target configured in our project file.

<i>If you are unfamiliar with targets, intuitively, they are different ways to build your application. Two targets in a single projects share most of their code and have roughly the same configuration, but have some differences. Targets are commonly used for separating “lite” and “premium” versions of an application, or even the iPhone from the iPad version. For more information, please refer to <a href="https://developer.apple.com/library/ios/featuredarticles/XcodeConcepts/Concept-Targets.html">Apple's documentation</a>.</i>

So, we have a target called “Dev Monologue” that is used to build the application itself, and another one called “Dev MonologueTests” that deals with unit testing. The first one contains (or will eventually contain) all “production” source files in the project – all the functionality needed for the application we are developing to work as intended. The second target should contain all test files, mock objects (classes created to assist testing) and classes that are subject to testing. Notice that the classes that are going to be tested and not necessarily all classes in the production code. You might decide not to test some modules, though it would be better if you did. For instance, the user interface is famous for being difficult for TDD because it is very subjective – how can you define that a certain component is in the right place or visualizes properly? That being said, there are many properties of your UI that can be unit tested. You can check if a button fires the right even when it receives a touch event, or that a view controller is popped from the navigation stack once the user dismisses it. Of course, you still need manual testing in order to be sure everything works fine. Also, as far as user interfaces are concerned, Xcode provides a handy tool for automating it – UIAutomation. I even wrote an article about it <a href="http://devmonologue.com/ios/debug/ui-automation-in-xcode/">here</a>.
<h2>Running Unit tests</h2>
Normally, when having several build target, one would choose which one to run. But in this case, it's a little bit different. You cannot select the testing target from the toolbar. Instead, you run your application normally by selecting “Run” from the menu (or CMD+R) and you start your unit test with Product → Test, or using the CMD+U shortcut.

[caption id="attachment_115" align="aligncenter" width="605"]<a href="{{ site.url }}/images/2014/02/tddStartTests.png"><img class="size-large wp-image-115" alt="The image shows the Product menu in Xcode expanded, exposing the Test option that is used for running the unit tests" src="{{ site.url }}/images/2014/02/tddStartTests-1024x642.png" width="605" height="379" /></a> Starting unit tests in Xcode[/caption]

Now let's see what happens when we run our new project:

[caption id="attachment_113" align="aligncenter" width="605"]<a href="{{ site.url }}/images/2014/02/tddFail.png"><img class="size-large wp-image-113" alt="The image shows the Xcode window just after running all unit test. It displays one failed test in the issue navigator and some comment inline in the source code editor" src="{{ site.url }}/images/2014/02/tddFail-1024x621.png" width="605" height="366" /></a> Unit testing results in Xcode[/caption]

Our testing fails. That's because Xcode's template created a single test for us that just instructs the OCUnit framework to fail, saying that we haven't implemented any unit tests yet. But that gives me the opportunity to explain how Xcode's TDD integration works. As you can see, failures in any unit test shows up in the issue navigator, just like compiler errors and warnings do. Which test failed and why is also described in the message. Additionally, most of this information is displayed inline in the source code editor, making it easy to see which line in the code caught the error.

That's right about it. That's all you need to know about the OCUnit integration in Xcode in order to start using it. You write code, press CMD+U, see what fails, fix it and repeat until all tests pass.
<h2>Anatomy of a unit test</h2>
The last section of this article is devoted to the syntax of writing unit tests with the OCUnit framework. It will not teach you to write meaningful tests, but rather, it will explain the mechanics of a test case class.

Since you already know how to setup a test driven development environment with Xcode, it is now time to replace the class Xcode generated with some real unit tests.

All test case classes in OCUnit are derived from XCTestCase. If the test classes that you create are not XCTestCase subclasses, they will not be run. Also, all of your methods have to have no return value (void) and take no parameters:
<pre class="brush:objc">- (void)nameOfTheTest;</pre>
The two rules described above are the only conditions that have to be met in order for your functions to be executed as unit tests. If you wonder how is this possible, the answer is that OCUnit uses Objective-C Runtime to get all XCTestCase subclasses and all of their methods that classify as tests.

Every unit testing class should contain two special functions – setUp and tearDown.
<h4>SetUp:</h4>
SetUp is called <b>before</b> the beginning of every unit test. It contains code, common to all tests in the class, that performs some initial configurations in order to prepare the environment for the validations that will follow. These include initializing the objects that are going to be tested, setting parameters and variables.
<h4>TearDown:</h4>
TearDown is called <b>after</b> each unit test in order to reset the environment and clean up. If some network connection have been initiated, for instance, this method might cancel and close the socket to avoid callback from being called during a subsequent test.

Note, that each test is executed independently from all others. Before it, all variables and states are reset in order to avoid side effects from another test to influence the next one.

So what does an actual test method look like? Well, let's look at what it has to accomplish. First of all, it has to create a setup for testing a particular scenario. Consequently, before anything else, the method has to initialize some variables in accordance to the desired functionality. After that, it needs to execute the code it is trying to verify. And lastly, the test needs to compare the result to a pre-calculated answer in order to determine whether to fail or succeed. This is done via assert methods. If you are unfamiliar with assert methods, they are functions that check a condition and if it is not true, cause the program to terminate. OCUnit provides the following assertion methods:
<ul>
	<li>XCTFail(format...)</li>
	<li>XCTAssertNil(a1, format...)</li>
	<li>XCTAssertNotNil(a1, format...)</li>
	<li>XCTAssert(expression, format...)</li>
	<li>XCTAssertTrue(expression, format...)</li>
	<li>XCTAssertFalse(expression, format...)</li>
	<li>XCTAssertEqualObjects(a1, a2, format...)</li>
	<li>XCTAssertNotEqualObjects(a1, a2, format...)</li>
	<li>XCTAssertEqual(a1, a2, format...)</li>
	<li>XCTAssertNotEqual(a1, a2, format...)</li>
	<li>XCTAssertEqualWithAccuracy(a1, a2, accuracy, format...)</li>
	<li>XCTAssertNotEqualWithAccuracy(a1, a2, accuracy, format...)</li>
	<li>XCTAssertThrows(expression, format...)</li>
	<li>XCTAssertThrowsSpecific(expression, specificException, format...)</li>
	<li>XCTAssertThrowsSpecificNamed(expression, specificException, exception_name, format...)</li>
	<li>XCTAssertNoThrow(expression, format...)</li>
	<li>XCTAssertNoThrowSpecific(expression, specificException, format...)</li>
	<li>XCTAssertNoThrowSpecificNamed(expression, specificException, exception_name, format...)</li>
</ul>
They should be fairly self explanatory so I'm not going to go into details on each of them. I'll just mention that all of the method above allow programmers to provide a human readable message that will be displayed whenever the assertion fails. The syntax is the same as with NSLog. In fact, we already saw the XCTFail method in action – it was a part of Xcode's template:

<code>
<span style="color: #643820;"><span style="font-family: Menlo-Regular, serif;">XCTFail</span></span><span style="color: #000000;"><span style="font-family: Menlo-Regular, serif;">(</span></span><span style="color: #c41a16;"><span style="font-family: Menlo-Regular, serif;">@"No implementation for \"%s\""</span></span><span style="color: #000000;"><span style="font-family: Menlo-Regular, serif;">, </span></span><span style="color: #aa0d91;"><span style="font-family: Menlo-Regular, serif;">__PRETTY_FUNCTION__</span></span><span style="color: #000000;"><span style="font-family: Menlo-Regular, serif;">);</span></span>
</code>

It caused our tests to fail and show the following explanation:

No implementation for "-[Dev_MonologueTests testExample]"

Putting everything in this section together, we can write a quick sample of how a unit test would look like using OCUnit:
<pre class="brush:objc">@implementation StringAppendTests

- (void)setUp
{
    [super setUp];
    firstString = @"one";
}

- (void)tearDown
{
    firstString = nil;
    [super tearDown];
}

- (void)testNSStringAbleToAppendString
{
	NSString* secondString = @" two";
	NSString* appendString = [firstString stringByAppendingString:secondString];
	XCTAssertEqualObjects(appendString, @"one two", @"NSString is not able to append strings");
}

@end</pre>
This example creates a new class called “StringAppendTests”. It is always good to name your test case classes in a way that hints what functionality it will check – in this sample – appending strings. Of course, there are many ways to append two strings and this class would probably check all of them – a test case class is by no means limited to a single test. The two special methods, setUp and tearDown, are really simple – they make sure the “firstString” variable is initialized. It is not mandatory to set the pointer to nil in the tearDown function – it will work fine without it, but it is included in the example as a reminder that you have to clean up after each test. Finally, let's look at the actual unit test. As already discussed, it has to have no return value and take no parameters. It is a good practice to have a potentially very long, but descriptive name for it, so that readers can easily understand what kind of scenario it is checking. The method's body holds no surprises either. It first initializes a second string variable, which is then appended to the first one. The last line check whether the result from the appending is the expected string “one two”.
<h2>Conclusion</h2>
Congratulations! You now know how to use the OCUnit framework in Xcode. As you can probably see, it is really simple and almost everything is already configured by the new project wizard. Introducing test driven development to existing project can be a little more difficult and I will probably cover that in a separate post.

So that's it for the second part of the Test Driven Development series. I hope you found it useful and you will stay tuned for the <a title="Writing Objective-C unit tests" href="http://devmonologue.com/ios/test-driven-development/writing-objective-c-unit-tests/">third part</a> where we will deep deeper into OCUnit and discussed some more advanced unit testing concepts.

<strong>Up next:</strong>
<ul>
	<li><a title="Writing Objective-C unit tests" href="http://devmonologue.com/ios/test-driven-development/writing-objective-c-unit-tests/">Part 3: Writing Objective-C unit tests</a></li>
</ul>
Once again, thanks for reading!