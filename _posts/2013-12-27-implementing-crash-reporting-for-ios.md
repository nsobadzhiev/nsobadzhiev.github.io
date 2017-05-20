---
id: 62
title: Crash reporting in iOS
date: 2013-12-27T18:12:27+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=62
permalink: /ios/implementing-crash-reporting-for-ios/
categories:
  - error-handling
  - iOS
tags:
  - backtrace
  - crash
  - ios
  - reporting
---
<h3>Introduction</h3>
We all work hard to prevent any crashes in our programs, but there's always some condition where the unexpected happens. Crashes in production might be a nightmare but they do happen. So for any application out there, whose developers are committed to resolving all imperfections in their product, crash reporting is critical. Crash reports can be seen and managed from Xcode's Organizer or in iTunes-Connect, but what if you wanted to receive then from anybody using the application? What if you wanted the application to automatically gather stack traces? We are going to do just that in this article.
<h3>What is a crash log?</h3>
A crash log is nothing more than a piece of text that indicates the contents of the program's stack (all functions being called by the application at a particular time) when the execution had to be stopped by the system. Many iOS developers think that these log can only be accessed by the system and they can only view them in Xcode's Organizer. However, the application has access to the data written in the program stack, provided that you know where to look at.
<h3>Why does an application crash</h3>
Many people don't exactly know what happens when an application crashes, so I decided to explain briefly. Most often, an application crashes because it violates a restriction imposed by the operating system. Let's say you try to access a memory that has already been released/deleted. The system realizes that you are trying to read or write data at an address that is no longer at the program's disposal (The OS knows which part of the memory it reserved for every running application so it knows when a program is trying to access parts of it that are outside it's authority). Since that operation is either a mistake in the application's logic, or malicious behavior, the operating system has no choice but to terminate the execution of the program.
<h3>Exit codes</h3>
When an application terminates, it returns a number that indicates how the process ended. As a convention, if the program exited successfully, it returns code zero. Every non-zero code means that the program failed at some point of it's execution. iOS is no exception - every time your application crashes, Xcode indicates the code it exited with. You have, no doubt seen them, though you might not have known they were exit codes. I'm talking about SIG_ABRT, BAD_ACCESS, SIG_KILL and maybe SIG_PIPE. These are all values that are trying to tell you what happened with your program. SIG_ABRT is most commonly associated with exceptions being thrown and not caught. SIG_KILL you can try by killing your application from the multitasking screen while attached to the debugger - it mean that your application was forcefully quit by another proces - normally it is not something you should be concerned with. BAD_ACCESS happens when you access memory that's already been deallocated. SIG_PIPE is kind of rare, but it means that you were trying to write into a pipe (a connection to a file or a server for instance) that has been closed by the remote party.
<h3>Signals</h3>
Signals are a method of inter process communication (IPC) where different programs running on the same system exchange information by sending each other number codes. Nowadays, it is almost exclusively used by the operating system in order to manage application programs (here, "application" refers to a program that is not part of the operating system - the kernel itself). The standard values are specified in the signal.h file in the iOS framework (of any UNIX framework). Signals can be intercepted by the program and handled (more on that below).
<h3>Accessing crash logs from an iOS application</h3>
How can an application access it's crash logs when the system terminates it? Well, this has something to do with those signals I was writing about above. Thing is...I lied a little. The operating system does not just terminate your application - it send a signal to it. Termination is just a side effect - the default action when receiving most signals, is for an application to crash (exit with non-zero code). You might already see where I'm going with this - we are going to replace the signal's default action with a custom one that extracts the crash log...

Let me step back for a little bit. As I mentioned, signals can be handled by programs. The way that works is, every signal is associated with a piece of code (a function) that is invoked to handle it. When the application starts, the default handlers are set in place. Most of them just make the application crash. However, at runtime, programs have the ability to replace these handler with a new one. So let's go ahead and do just that.
<h3>Setting signal handlers</h3>
Finally, let's write some code. Since signals are deep in the C libraries, fiddling around with them requires some C/C++ magic.

To use the following examples, you will need to import some headers:
<pre class="brush:objc">#import &lt;signal.h&gt;
#import &lt;execinfo.h&gt;</pre>
<h4>First of all, let's create our signal handling function:</h4>
<pre class="brush:cpp">void crashSignalHandler(int signal)
{
    // do magic here...
    exit(signal);
}</pre>
As you can clearly see, it is nothing special - it's just a standard void C function with one parameter of type int where it receives the signal code it has to handle. Any function that fulfills these requirements can be a signal handler. I should emphasis that it cannot be a method of a class. What's interesting here is the last line in the function - <code>exit(signal);</code>. It is actually kind of important - it is going to terminate our application once we handle the signal and before we exit the function. WHAT? Why would I want to crash my own application since I have a chance to save it. Well, I thought the same thing, but once you receive a signal like BAD_ACCESS, even if you don't call the exit function, the application will freeze. It is better to just let go sometimes...and let your creation crash (and burn). Please note that because we replace the default handler (that makes the application crash) with our own, it no longer exits automatically and that's why it will not crash unless we call the exit function.
<h4>Replace the default signal handler:</h4>
We wrote a function to handle our signal, but it doesn't get called. We have to instruct the system to remove the default handler and start using ours. Here's how we do that:
<pre class="brush:cpp">signal(SIGABRT, crashSignalHandler);</pre>
This will set a new handler for the SIGABRT signal. However, not all crashes are produced by SIGABRT. In fact, these are relatively easy to fix. What we need is to intercept BAD_ACCESS.
<pre class="brush:cpp">signal(SIGABRT, crashSignalHandler);
signal(SIGSEGV, crashSignalHandler);
signal(SIGBUS, crashSignalHandler);</pre>
I found that most crashes are caught with these three signals.
<h4>Revert to the default signal handler:</h4>
You might want to disable your custom crash reporting in some occasions. To achieve this, you have to remove your handler and put the default one in place.
<pre class="brush:cpp">signal(SIGABRT, SIG_DFL);
signal(SIGSEGV, SIG_DFL);
signal(SIGBUS, SIG_DFL);</pre>
This one was easy. SIG_DFL is just a #define for the default function.
<h3>Retrieving the stack trace</h3>
So now, the only thing left to do is to save the stack trace in our handler. Since the application is going to terminate right after the handler returns, we are going to save the log in the file system so we can access it next time the application is run. To read the stack trace we will use another C magic:
<pre class="brush:cpp">void* callstack[128];
int frames = backtrace(callstack, 128);</pre>
This will copy the first 128 entries in the program's stack into the "callback" array. To desymbolicate it and save it into a file, we can use the following:
<pre class="brush:cpp">backtrace_symbols_fd(callstack, frames, crashLogFileDescriptor);</pre>
The "backtrace_symbols_fd" function will take the array we received above and write it to a file. It takes the file descriptor of the target file, so you will have to open it yourself and get the file descriptor. If you don'r already know, a file descriptor is an integer that identifies an open file within the file system. You can acquire it by opening the file in the following way:
<pre class="brush:objc">const char* fileNameCString = [crashLogFilePath cStringUsingEncoding:NSUTF8StringEncoding];
FILE* crashFile = fopen(fileNameCString, "w");
crashLogFileDescriptor = crashFile-&gt;_file;</pre>
The resulting file will contain something like this:
<pre class="brush:xml">0   AppName                             0x00206077 crashSignalHandler + 306
1   libsystem_platform.dylib            0x3aa7005b _sigtramp + 34
2   libsystem_pthread.dylib             0x3aa75a33 pthread_kill + 58
3   libsystem_c.dylib                   0x3a9bcffd abort + 76
4   libc++abi.dylib                     0x39cebcd7 &lt;redacted&gt; + 74
5   libc++abi.dylib                     0x39d046e5 &lt;redacted&gt; + 252
6   libobjc.A.dylib                     0x3a44d921 &lt;redacted&gt; + 192
7   libc++abi.dylib                     0x39d021c7 &lt;redacted&gt; + 78
8   libc++abi.dylib                     0x39d01d2d __cxa_increment_exception_refcount + 0
9   libobjc.A.dylib                     0x3a44d7f7 objc_exception_rethrow + 42
10  CoreFoundation                      0x2ff40c9d CFRunLoopRunSpecific + 640
11  CoreFoundation                      0x2ff40a0b CFRunLoopRunInMode + 106
12  GraphicsServices                    0x34c67283 GSEventRunModal + 138
13  UIKit                               0x327e4049 UIApplicationMain + 1136
14  AppName                             0x000cd07f main + 154
15  AppName                             0x00008c28 start + 40</pre>
System method show as "redacted", but you should be able to see functions that you wrote for your application.

Here's the whole handler function:
<pre class="brush:objc">void crashSignalHandler(int signal)
{
    NSlog(@"Application received signal: %d", signal);
    void* callstack[128];
    int frames = backtrace(callstack, 128);
    backtrace_symbols_fd(callstack, frames, crashLogFileDescriptor);
    exit(signal);
}</pre>
<strong>Note: </strong>You can read the manual page for all functions mentioned above type typing "man &lt;functionName&gt;" in the Terminal application:
<pre class="brush:shell">man backtrace_symbols_fd</pre>
<h3>Conclusion</h3>
So there you go...crash reporting in iOS applications. Now you wont wonder how they did it next time you see and alert asking you if you wanted to send a crash log to the developers of an application you use. As you can see, it's nothing hard, you just need to know where to look. Let me know in the comment section - Will you be implementing crash reporting for your app, and if you have, was it similar to the way I showed you?

Once again, thanks for reading!