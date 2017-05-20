---
id: 38
title: Using the debugger console lldb in Xcode
date: 2013-12-15T11:03:41+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=38
permalink: /ios/using-the-debugger-console-lldb-in-xcode/
categories:
  - debug
  - iOS
tags:
  - debug
  - ios
  - Xcode
---
&nbsp;

Let's take a moment to talk about the Xcode debugger. I'm sure each and every one of you knows how to debug, but here, we are going to be talking about some "more advanced" usage of lldb in Xcode. This post is not going to be about continuing or stepping over or setting a breakpoint - it's about getting more information from the program while we are at a breakpoint. These are features of the lldb debugger that I use on daily bases and I thought I'd share my experience.
<h3>Using lldb in Xcode</h3>
This one's easy - you probably already know how to do that, but let's mention it real quick just for reference. The debug console is located on the bottom of the console area in Xcode:

<a href="{{ site.url }}/images/2013/12/debugConsole.png"><img class="size-medium wp-image-39 aligncenter" alt="debugConsole" src="{{ site.url }}/images/2013/12/debugConsole-300x75.png" width="300" height="75" /></a>

&nbsp;

Typing commands there is done by clicking next to the blue <span style="color: #3366ff;">(lldb) <span style="color: #000000;">text. When you do that, a cursor will appear on the right and if you start typing, text should appear on the screen like in the screenshot above.</span></span>
<h3>Printing backtraces in the lldb console</h3>
Have you ever received crash reports from another developer? When they sent you a back trace, did it look like this:

<a href="{{ site.url }}/images/2013/12/debugbt.png"><img class="size-medium wp-image-40 aligncenter" alt="debugbt" src="{{ site.url }}/images/2013/12/debugbt-262x300.png" width="262" height="300" /></a>

If this has happened to you... I feel your pain. It pains me to see something like this. The RIGHT way to get a backtrace while debugging is by typing the following:

<strong><span style="color: #3366ff;">(lldb) </span><span style="color: #000000;">bt</span></strong>

The bt command (short for backtrace) will print the current thread's stack in the console. You can then copy it and send it to a colleague. Sure beats launching the Grab application to make a screenshot.
<h3>Printing objects in the lldb console</h3>
While you are at a breakpoint, we can see the values of all variables in the current scope by looking at the debugging area in Xcode. This is extremely helpful, but you will agree that it doesn't always show what you're looking for. Consider this - you have some NSData coming from an NSURLConnection and you want to see what's written there. If you try to do that in the debugging area, you will only see some addresses and hex values. You'd much rather see it as a string. Even the "print description" option will be useless in this case. You need to get a string representation of your data - you need

<code>[[NSString alloc] initWithData:data:usingEncoding:NSUTF8StringEncoding]</code>

We are going to do just that using lldb. The "po" (print object) command takes an arbitrary expression, that returns an object and prints it's description in the console. So the following statement:

<strong><span style="color: #3366ff;">(lldb)</span></strong> <strong>po </strong><code><strong>[[NSString alloc] initWithData:data:usingEncoding:NSUTF8StringEncoding]
</strong></code>

Is going to print the string representation of data. Well, actually, it will print:

<code><b><span style="color: #3366ff;">(lldb)</span> </b><b>po [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]</b>
error: use of undeclared identifier 'NSUTF8StringEncoding'
error: 1 errors parsing expression</code>

Unfortunately, lldb doesn't seem to be able to recognize enums. So, instead of using NSUTF8StringEncoding, we will have to use it's integer representation - 4.

<code><b><span style="color: #3366ff;">(lldb)</span> </b><b>po [[NSString alloc] initWithData:data encoding:4]</b>
I love chicken!</code>

So that's printing objects covered. But not everything in Objective-C is an object. You also have primitive data like int or char for example. What's more, C++ objects are not the same as Objective-C objects. How do you print those?
<h3>The lldb print command</h3>
The print command can be used to print primitive data types. It's syntax is the same as po:

<code><b><span style="color: #3366ff;">(lldb)</span> </b><b>print array.count</b>
5</code>
<h3>Printf in lldb</h3>
When it comes to printing debug information, you can't beat printf. For example, I use it whenever I want to print a C++ object. Of course, it needs to have a method like "toString" that can generate a string representation of it. Here's how you do it:

<code><b><span style="color: #3366ff;">(lldb)</span> </b><b>print (int) printf("%s", cObject-&gt;toString())</b>
</code>

So printf allows you to display any information (currently in scope) that you can using the printf command.

Another trick I use with Xcode's debugger is that when I want to extract some application-wide information, I pause the program's execution using the "Pause" button in the debug area and then use "po" or "print" to get some data. For example, I needed to extract the token used for communicating with a server. It was stored in a singleton object named LoginManager. I was able to copy the access token using the following command:

<code><b><span style="color: #3366ff;">(lldb)</span> </b><b>po [[LoginManager instance] token]</b>
</code>
<h3>gcc vs. lldb</h3>
I know some of you might already know how to use the gcc console and are transitioning into lldb.  <a href="http://lldb.llvm.org/lldb-gdb.html">This table</a> will show you all gcc commands and their lldb counterparts.
<h3>Conclusion</h3>
It might not seem like that at first, but once you start using these lldb commands (and maybe others) you will notice that they can help a lot. They are a powerful tool in any developer's repertoire. I suggest you give it a shot and see for yourself.

Once again, thanks for reading.