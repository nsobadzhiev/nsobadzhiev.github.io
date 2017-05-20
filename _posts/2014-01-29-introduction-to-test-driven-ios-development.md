---
id: 87
title: Introduction to Test Driven iOS development
date: 2014-01-29T20:11:44+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=87
permalink: /test-driven-development/introduction-to-test-driven-ios-development/
categories:
  - Test Driven Development
tags:
  - TDD
  - Test Driven Development
---
<i>This post will be the first part of a TDD series. It will guide you in the process of integrating test driven development in your projects. In this first article, we will be discussing what test driven iOS development is and how it can be used when developing for Mac or iPhone/iPad.</i>
<h3>What is Test-Driven Development (or TDD)?</h3>
Test-Driven Development is a section of extreme programming where programmers write tests for the code that they are developing. These tests should prove that the code is working correctly in a given circumstance. In the purest form of TDD, these test are written before the real (production) code. This forces developers to first think about how their code is going to be used before they even attempt to write it. It ensures that a new functionality is written according to the requirements, and not according to the details of implementation. Also, it ensures that functionality that is not going to be used, is not added in the first place<sup>[1]</sup>. But why is this important? What if it's not used – someday it might be? Well, that's exactly the problem. Since this functionality is not used at first, it is probably not tested well enough and might not be working properly. And when, later on, someone decides to use it, he/she will run into problems. This problem is often referred to as YAGNI (Ya Ain't Gonna Need It).

Another core concept in test driven development is the so called “Red, Green, Refactor”. It refers to the process you follow while you are using TDD.

<strong><span style="text-decoration: underline;">Red:</span></strong>

You write a new test that proves that the next functionality you plan on implementing is working. At this point, if you run your code, this test will fail, because you don't have that functionality yet. Nevertheless, you should do it to prove that the test does not pass with the current code. If you wonder, this stage is called “Red” because when you run the project, your IDE will flash red due to the failing test.

<strong><span style="text-decoration: underline;">Green:</span></strong>

After the first stage is complete, you are ready to write the code that actually implements the feature. It is important to write just the bare minimum that will make the test pass. You don't need to provide a complete implementation. For example, lets say you have to implement a method that multiplies two numbers. You are testing this new method by calling it with arguments 2 and 4. Using the “Red, Green, Refactor” technique, it is perfectly fine to implement a function that just returns 8 (return 8;). This is the absolute minimum needed in order for the test to pass. It might seem insane at first, but that's how test driven development works. The idea here is that after this test, you will create another one (lets say that it will test if 0 * 13 = 0 to be sure that our implementation handles zero correctly) and at that point our initial implementation of the method will no longer be able to pass the test and we will be forced to replace it with a real one. This concept introduces incremental changes in your code. With every new test that you add, another small change is introduced in the production code.

<strong><span style="text-decoration: underline;">Refactor:</span></strong>

You might wonder - “If I make all those incremental and minimalistic changes to my code, without ever thinking about the big picture, wont my code be doomed to have poor structural properties?”. That's what I thought anyway. Well, it is true that thinking only locally might result in a code that is not all that good. But that's where the “Refactor” stage come in handy. Once you complete the “Red” and “Green” stages, it is time to think about what you've created, see where you can improve. Maybe you realized that it would be better to extract something into a method or there is a better way to implement something. If you do, you are free to makes whatever refactoring you want (as long as it is covered by the tests). And since you have all those tests that verify that your code is still working, refactoring should be safe. If you introduced some regressions, testing would fail and you will be able to fix your mistake before it causes any harm.

This is the most powerful benefit of using test driven development – you no longer have to fear that if you change a piece of code, something else will break. Of course, it does not guarantee a bug free software. In fact, code is only as good as its tests. If your tests have good coverage and check all possible error scenarios, you might be relatively confident that the code is working properly. However, you certainly wont have the time and resources to write tests for everything. There is a trade-off between how much you want to ensure your application works and how much are you willing to “pay” for it. It all depends on what your project is and whatever works for you. Intuitively, you should always write more tests for code that is critical to your application and is vulnerable to regressions and fewer tests in places that you feel confident.

If my calculations are correct, you should be starting to wonder if test driven development is actually worth it. It certainly seems that it slows you down a lot. And you end up writing much more code. Well, it is true, and I cannot be sure how it compares to conventional programming techniques. All I know is that the experts say it <b>does </b>pay off in the end. It's supposed to save 20% (as far as I remember) development time when considering time for bug fixes. As you should know, the sooner a bug is discovered, the cheaper it is to fix, and since you don't have to do as much bug fixing upon completion, if a project is developed using TDD, the overall process becomes more efficient. My personal opinion is that, if quality is important to you as a developer, and there is not much pressure to complete a project, test driven development is always worthwhile.
<h3>Test driven development on existing projects</h3>
Lastly, lets discuss test driven development on existing projects. Not everybody has the luxury of starting a project from scratch. In such case, it looks as if TDD is out of the question. It maybe is, in it's purest form, but you can still start testing your code. The first way to do it that comes to mind is to start writing tests for all currently available code. Well, that's generally not a good idea, because, first of all, it violates the principal of writing tests first. But the more important consideration is that you lose the perspective that TDD gives you while you write. As we discussed previously, TDD forces you to think about requirements, rather than the details of implementation while you are writing a new feature. And if you start writing a bunch of test for existing code, you will be designing them around your implementation, whereas it should be the other way around. What good is it to start writing test that are designed so that they pass. And last, but certainly not least, writing tests for <b>all </b>existing code in the project would be terribly boring. :)

So what is the right way to start using test driven development in an existing project? The answer is that you will have to start from somewhere – just write tests for all new features and fixes that you develop in the future. That would mean that some parts will use TDD and others wont, but that's the price you will have to pay.

Unfortunately, even like that you will most likely run into problems unless your existing project is really well written. Test driven development requires code with little dependancies – in order for a unit test to check only one functionality, the production code that implements it will have to do that (and only that) – hence the Single Responsibility Principle has to be in place. This is often not the case.

In order to fix that, you will have to break the <i><a href="http://www.webopedia.com/TERM/T/tight_coupling.html">tight coupling</a> </i>and start testing the new code. A common technique is to think if a place where the parts of your application can be separated (for instance, the line between the controller and the view is a Model-View-Controller environment like Cocoa). Think if the classes that sit on either side of that line and revise their functionality for something that does not belong and can be moved. Do this until you are satisfied that everything is in it's right place. If you have done this properly, this would be the place you could start using TDD.

<i>I think that's enough for the first part of the Test-Driven-Development series. We got acquainted with TDD and we covered some core principles and techniques so that you know what you are getting into. This description was by no means complete – there is still a lot to learn. If you are interested, I will probably add some links to additional resources.</i>

<i>Next time we will be covering several frameworks for test driven development in iOS that will guide you to some specifics in introducing TDD for your Xcode projects.</i>
<ul>
	<li><strong>Up next:</strong></li>
	<li><a title="Test Driven Development with OCUnit" href="http://devmonologue.com/ios/test-driven-development/test-driven-development-ocunit/">Part 2: Test Driven Development with OCUnit</a></li>
	<li><a title="Writing Objective-C unit tests" href="http://devmonologue.com/ios/test-driven-development/writing-objective-c-unit-tests/">Part 3: Writing Objective-C unit tests</a></li>
	<li></li>
</ul>
Thanks for reading!

[1.] How many times have you written something because you thought it could be useful, but when integrating your code, discovered that you don't actually need it? TDD makes sure you never end up in a situation like this, because you already know how you're going to be using your code.
<h3>References</h3>
<ol>
	<li><a href="http://www.sunetos.com/items/2011/10/24/tdd-ios-part-1/">http://www.sunetos.com/items/2011/10/24/tdd-ios-part-1/</a></li>
	<li><a href="http://www.amazon.com/Test-Driven-iOS-Development-Developers-Library/dp/0321774183/ref=sr_1_1?s=books&amp;ie=UTF8&amp;qid=1391082363&amp;sr=1-1&amp;keywords=ios+test+driven+development">iOS Test driven development -- Graham Lee</a></li>
</ol>
	<link href="http://purl.org/dc/elements/1.1/" rel="schema.DC" hreflang="en" /> 	<link href="http://purl.org/dc/terms/" rel="schema.DCTERMS" hreflang="en" /> 	<link href="http://purl.org/dc/dcmitype/" rel="schema.DCTYPE" hreflang="en" /> 	<link href="http://purl.org/dc/dcam/" rel="schema.DCAM" hreflang="en" />

<style type="text/css"><!--
@page {  }
table { border-collapse:collapse; border-spacing:0; empty-cells:show }
td, th { vertical-align:top; font-size:12pt;}
h1, h2, h3, h4, h5, h6 { clear:both }
ol, ul { margin:0; padding:0;}
li { list-style: none; margin:0; padding:0;}
<!-- "li span.odfLiEnd" - IE 7 issue-->
li span. { clear: both; line-height:0; width:0; height:0; margin:0; padding:0; }
span.footnodeNumber { padding-right:1em; }
span.annotation_style_by_filter { font-size:95%; font-family:Arial; background-color:#fff000;  margin:0; border:0; padding:0;  }
* { margin:0;}
.P1 { font-size:12pt; font-family:Times New Roman; writing-mode:page; }
.P10 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; text-decoration:none ! important; }
.P11 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; text-decoration:none ! important; }
.P12 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; text-decoration:none ! important; }
.P13 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; text-decoration:none ! important; }
.P14 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; text-decoration:none ! important; font-weight:bold; }
.P15 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; text-decoration:none ! important; font-weight:normal; }
.P16 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; text-decoration:none ! important; font-weight:normal; }
.P17 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; font-weight:bold; }
.P18 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:italic; text-decoration:none ! important; }
.P2 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:italic; }
.P3 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; }
.P4 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; }
.P5 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; }
.P6 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; }
.P7 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; text-decoration:underline; }
.P8 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; text-decoration:underline; }
.P9 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; text-decoration:none ! important; }
.T10 { font-weight:normal; }
.T11 { font-weight:normal; }
.T12 { font-weight:normal; }
.T13 { font-style:italic; font-weight:normal; }
.T15 { vertical-align:super; font-size:58%;}
.T3 { text-decoration:underline; }
.T7 { font-weight:bold; }
.T8 { font-weight:bold; }
.T9 { font-weight:normal; }
<!-- ODF styles with no properties representable as CSS -->
.Endnote_20_Symbol .T1 .T14 .T17 .T2 .T4 .T5 .T6  { }
--></style>