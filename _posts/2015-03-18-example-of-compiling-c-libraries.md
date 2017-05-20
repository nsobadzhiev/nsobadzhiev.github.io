---
id: 371
title: Example of compiling C libraries
date: 2015-03-18T12:23:13+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=371
permalink: /using-libraries/example-of-compiling-c-libraries/
categories:
  - Using libraries
tags:
  - configure
  - header search paths
  - ios
  - libraries
  - Library search paths
  - make
  - Makefile
  - Objective-C
  - static libraries
---
## Previously on DevMonologue.com

In my [last article](http://devmonologue.com/ios/using-libraries/compiling-libraries/) we looked at how you can compile C libraries written by other people and integrate them into your Xcode project as static libraries. It covered some basics regarding the build process of many standard open source libraries. However, often times things don’t exactly go according to plan and an inexperienced developer can be left with cryptic compiler and linker errors.

## And now... an example of compiling C libraries

So today we are going to look at an example of compiling C libraries. Seeing it in practice will be much more helpful and easier to understand. I’ll try to be as explicit as I can and make sure to explain everything I do. The whole process might prove a little scary to some of you, but you’ll be thanking me one day when you finally rub noses with such a problem.

So lets get on with it!

## The project

*All of the articles in this blog are about things that I either encountered in my programming career or I found interesting and wanted to learn more about. This post is no different, so the example of compiling C libraries we’re going to be working on, is something I build recently. To be honest, there are libraries that would be more useful to the majority of you, but nevertheless I believe the task I had shows many aspects of working with open source libraries that are a perfect project for this article*

Today’s example of compiling C libraries will be something called JusText. The JusText algorithm is designed to take a whole HTML webpage as input, and output the useful, human readable “payload”. It quite an interesting project, which is luckily ported to C++ by István Endrédy (thank you sir!). So for starters, we need to compile that on the Mac for the x86, x86_64, armv7, armv7s and arm64 architectures.

__Note:__ *What are all these architectures and why do I need them? Well, these are the currently “mainstream” architectures for iOS projects. Here’s a quick summary:*
* x86 - this is the standard desktop architecture from Intel that your Mac uses. For iOS apps it allows you to run your project on the simulator
* x86_64 - The 64-bit version of x86
* armv7 - The architecture that the iPhone (4S, I believe) uses
* armv7s - The architecture that slightly newer iPhones use (iPhone 5)
* arm64 - The 64-bit version of the ARM architecture (iPhone 5S and iPhone 6)

However, there’s a caveat to building JusText. It has dependencies:
* htmlcxx
* pcrecpp

And before you think I set you up, know that this is very common. Most libraries out there use other libraries. You cannot realistically expect them to write everything from scratch. So, before we are able to successfully build JusText, we will have to compile htmlcxx and pcrecpp. But what are these projects?

__htmlcxx__ is used to parse HTML. Note that you cannot just use an XML parser since HTML is often not valid XML and parsing will just fail.

__pcrecpp__ is a C++ library that provides regular expression support.

To summarize, our task consists of the following:
* Compile pcrecpp into a static library
* Compile htmlcxx into a static library
* Compile JusText using the libraries above
* Import JusText into your iOS project and start using it!

# The plan

Looking at the source code of the libraries you wish to compile, there are two way to go. The first way is the one I showed you in my [previous article](http://devmonologue.com/ios/using-libraries/compiling-libraries/) - using it’s Make script. The other one is to compile the sources yourself using Xcode. There’s nothing stopping you from creating a new project, adding all files and compiling them as if you would with your own source code. This approach works well for small libraries that don’t need any special build configuration. However, for larger projects, it can quickly become too complicated to setup the Xcode project and figure out all dependencies.

For the purpose of this tutorial, we are going to be using both methods. In fact, we cannot stick to only one of them. Here’s why:

Our final product - the JusText library, doesn’t have a Makefile or a build script. It only contains a Visual Studio project file. So we will have to create a custom Xcode project for it. But don’t worry, the sources themselves are not very complicated so it should be fairly straight-forward.

Now, to satisfy JusText’s dependencies, lets look at pcrecpp and htmlcxx.

With htmlcxx, the decision is trivial. JusText’s author has already opted to include it’s sources as part of his project. That means that instead of creating a static library for htmlcxx, we will be importing the sources and compiling them as part of the JusText project.

On the other hand, pcrecpp is not included in the project. We will have to satisfy that dependency ourselves before compiling JusText. Now, looking at the source code for pcrecpp, we can see that it DOES contain a Makefile. Additionally, it has multiple products (the pcre library, the pcrecpp library along with other things) so we better let the Makefile deal with creating them rather than try to replicate that with Xcode.

So, upon further investigation, our tasks get a little updated:
* Compile pcrecpp into a static library
* Setup Xcode project for JusText
* Add pcrecpp as a static library
* Compile JusText as a static library
* Import JusText into your iOS project and start using it!

## PCRE

### Compile pcrecpp into a static library

Those of you who have already read my [article about compiling C libraries](http://devmonologue.com/ios/using-libraries/compiling-libraries/), should already know how to do that. If not, you can should go ahead and read it before continuing.

But there is a problem. Following the steps in my previous article will build pcre for x86_64 (x86 if your Mac is not 64-bit). In order to build for the ARM architectures, we will need to modify our call to the configure script. More specifically, we need to instruct it to NOT use the default gcc and g++ compilers, but rather the ones that come with the iOS SDK. That’s because the gcc and g++ compilers that come with the SDK are specifically designed to build for iOS, meaning the ARM architecture in this case.

So, previously, we called the configure script without parameters:

``` bash
$ ./configure
```

But now... wait for it...

``` bash
$ ./configure --disable-shared --enable-utf8 --host=arm-apple-darwin CFLAGS="-arch armv7 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS8.1.sdk" CXXFLAGS="-arch armv7 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS8.1.sdk" LDFLAGS="-L." CC="/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/cc" CXX="/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/c++"
```

Yeah... So that’s a little different. Configure script parameters are way beyond the scope of a single article and I’m certainly not the man to explain them, but generally there are several things to note here.

Firstly, note that we explicitly instructed the configure script to build for the armv7 architecture using `-arch armv7`. Note that we can change that to `armv7s`, `x86`, `x86_64` and `arm_64`. In fact, we will do just that later.

The other parameters set the path to the C and C++ compilers that we want to be used during the build process. Additionally, we specify the path to the iOS SDK that we want to build with.

__Note:__ The file system paths that I’m using might not work for you if you’ve installed Xcode elsewhere or are building with different SDK. If you encounter any errors in the configure script, make sure that these paths are indeed correct.

If all goes well with the configure script, you can go ahead and run `make` to generate the library file.

At this point, you can go to the `.lib` folder and find all kinds of goodies:

```
libpcre.a libpcre.lai libpcrecpp.la libpcreposix.a libpcreposix.lai pcre_stringpiece_unittest pcregrep
libpcre.la libpcrecpp.a libpcrecpp.lai libpcreposix.la pcre_scanner_unittest pcrecpp_unittest pcretest
```

__Note:__ `.lib` is a hidden folder so you might not see it in Finder.

Out of these, we will need:

```
libpcre.a
libpcreposix.a
libpcrecpp.a
```

### Making a universal static library

You might notice that we only built the library for armv7. And I promised that it was going to be working with all the popular architectures we use for iOS development. The way we are going to achieve that might not the easiest or most elegant, but here it is. We are going to generate a static library file for every architecture we need by modifying our call to the configure script. It should be almost the same as the one shown above - just replace all references to `armv7` with the new architecture you want to build for - `arm64` for example. After each build, go ahead and copy the library files in a separate directory. Once you have one for every architecture you want, type the following in the Terminal:

``` bash
$ lipo -create /path/to/library1 /path/to/library2 /path/to/library3 -output /path/to/universal_library.a
```

So what just happened? We took several library files and created what’s called a fat library. Thats a library that contains binaries for several architectures.

### Preparing PCRE for use

Now that we have a pre-built version of the pcre and pcrecpp libraries, we are almost ready to check that off our list and continue with the next task at hand. So what’s still missing?

If you’ve read my previous article you might recall that in order to use a library, you will need two things - the library itself and its public headers. The headers, of course, are needed because the compiler doesn’t know about our library even after importing it into the project. Using the headers, we “promise” that we will provide implementation for the library method call during the linking process.

Normally, by invoking the `make install` script, the public header files are automatically copied into one of the standard header files folder - like `/usr/local/include/`. However, since we were being “clever” during the last step, we didn’t install our library and we will have to copy the headers ourselves.

One way to do that would be to open the Makefile and figure out which files it copies. However, since the PCRE library is relatively simple, you can kind of “guess” which files you’ll need to get from the source files. So we can get away with just copying all `.h` files from PCRE over to our future project file’s directory.

## JusText

Good news! We’ve just completed the hard part of this tutorial. Now all we have to do is create a new Xcode project and add our PCRE universal library to it.

As discussed above, we are going to download the source code for JusText from github, add it to out Xcode project, satisfy its dependencies and hopefully... build it.

### Setup a new Xcode project

First things first, lets download [JusText’s source code](https://github.com/endredy/jusText).

Next, in order to create an Xcode project for a static library, go to File -&gt; New -&gt; Project -&gt; Framework &amp; Library -&gt; Cocoa Touch Static Library. Once you have done that, go ahead and add all sources for JusText (excluding the Visual Studio files and the test directory).

![Project tree screenshot]({{ site.url }}/images/2015/03/JusTextProjectTree.png "Picture showing all JusText sources imported into the project")

As you can see, it already contains the sources for the htmlcxx library so we don’t have to worry about that. What we have to worry about is pcre. Lets add it to the project, alongside it’s header files.

![PCRE headers]({{ site.url }}/images/2015/03/PcreHeadersTree.png "Picture showing all PCRE public headers inside the project tree")
Now we are kind of ready to try and build JusText. But (**Spoiler alert**) we are going to run into a couple of problems. Go ahead and hit that Build button, I dare you!

### Fixing standard library errors

![Standard library error]({{ site.url }}/images/2015/03/StdCompileError.png "Picture showing an Xcode compilation error - 'bits/char_traits.h' file not found")

```
ci_string.h:8:10: 'bits/char_traits.h' file not found
```

Yeah... Now what? This error might seem a bit obscure... and it is. But a little googling around and you’ll find that Xcode doesn’t seem to be working that well with the C standard libraries sometimes. But here’s the clever bit - we are going change the C standard library the project is built with.

![Changing the standard library]({{ site.url }}/images/2015/03/ChangeCompiler.gif "A GIF showing how to add an entry in the C standard library field of the project settings")

Whaaaaat? I bet you thought you’d never have to change these project settings. But those kind of problems are to be expected whenever you are trying to work with C++ libraries. They tend to use stuff from the standard libraries and even though they are supported by Xcode, it’s not always the default setting.

Alright! Let try building again. Surely everything will be fine this time.

![ParserDom error]({{ site.url }}/images/2015/03/ParserDomError.png "A picture of the 'html/ParserDom.h' file not found compiler error")

```
justext.h:7:10: 'html/ParserDom.h' file not found
```

### Setting header search paths

Oh man, not again! Well, get used to it. You are bound to run into these sort of problems. So what happened here? The compiler cannot find `ParserDom.h`. But why? It’s right there in the project tree!

The problem is with the import statement.

``` c
#include <html/ParserDom.h>
```

If we change that to

``` c
#include "ParserDom.h"
```

the problem will be fixed. You would recall that the difference between these statements is that the latter checks the project directories for the specified file, whereas the former searches all the standard places your environment puts its headers.

That being said, changing the include statement is not best thing you can do in this situation. It’s not going to win you any respect with the developer society. That’s because both JusText and htmlcxx are third-party projects that you should avoid modifying unless you have something meaningful to add. It’s just cleaner that way.

We need another plan to fix this error. If we cannot change the include statement, we will have to change the places Xcode searches for headers. That’s done in the “Header search paths” field in the project settings. We’ll add `JusText/htmlcxx-0.84` in there and specify that Xcode should search in there recursively since the actual headers are one level deeper in the `html` directory.

![htmlcxx search paths]({{ site.url }}/images/2015/03/HtmlcxxSearchPath.gif "A GIF showing how to add a header search path for htmlcxx")

That should be enough to make this error go away. But the truth is that we are still not done... If you try to build once again, you’ll see that we get the same error for PCRE’s headers. At this point you should know how to handle that. Just for the sake of clarity, you need to add `JusText/libraries/pcre/headers/` to the search path. This time you don't need to make that path recursive because that’s the exact path to the header files.

Before you start hating me for making you work with header search paths, please consider that this, along with library search paths, is a big part of working with third-party sources and workspaces. So you better get used to it.

**NOTE:** *Library search paths is the same as header search paths, but instead of telling the compiler where to look for headers, it tells the linker where to find libraries. If your library file is in a non-standard path, you may need to add its location in the library search paths. We didn’t do that for the JusText project because we added the PCRE static library in our project tree, thus including it in the build phases “Link binary using libraries” list*

OK, so now what happens when we hit that Build button. Will there be another error? No! At this point the project should compile and link fine! Well done, we finally have a result! If you look under “Products” in your project tree, you should even see your built static library. Isn’t it satisfying...?

## Valid architectures

After all that trouble we went through to build PCRE for many architectures, it makes sense to make sure our newly built JusText library does so as well. Unlike what we had to do previously, setting up Xcode to generate a binary valid for several architectures is quite simple.

First of all, in the “valid architectures” field of our project file, we will have to enumerate all architectures we need. In our case, that might be “armv7”, armv7s and “arm64”. In addition to that, we need to tell Xcode not only that all these options are acceptable build configurations for the project, but we also want it to include all these architectures in a single binary every time we build. That’s done by specifying “NO” in the “Build Active Architecture Only” field. However, it makes sense to leave that to “YES” for Debug builds in order to shorten build times.

![Valid architectures]({{ site.url }}/images/2015/03/ValidArchitectures.png "Picture showing the valid architecture section in the project settings")

## Finishing off

Wow... that was intense! Using third party libraries is hard work... Well, not as hard as writing that extra functionality yourself :)

### Sample projects

After all that talking, it is now finally time for me to show you some code. A completed Xcode project, containing everything we discussed here about the example of compiling C libraries is available on [my github page](https://github.com/nsobadzhiev/JusText_iOS).

Also, I created a simple console utility that utilizes our newly created static library. It is also [available on github](https://github.com/nsobadzhiev/JusTextUtil).

One thing you might notice in JusTextUtil is that it also contains all static libraries related to PCRE. This is no coincidence. In fact, it is the client of a static library’s responsibility to satisfy all dependencies. What’s more, you can safely remove “libpcre.a” from the JusText project and it will build fine. It is the JusTextUtl project where the the build process needs to resolve all method calls in order to create the binary.

I hope you enjoyed this example of compiling C libraries, found it useful and you don’t get terrified next time you need to work with libraries. If there is anything unclear about the project we just completed, be sure to leave a comment so we can work together on solving each other’s development problems. For more tutorials about using libraries consider subscribing to the DevMonologue blog via email or RSS. Thanks for reading!