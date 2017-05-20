---
id: 353
title: Compiling libraries
date: 2015-02-09T11:49:01+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=353
permalink: /using-libraries/compiling-libraries/
categories:
  - Using libraries
tags:
  - compile
  - framework
  - libraries
  - make
  - static library
  - Xcode
---
This is an article that will teach you how to download, compile and use a C (or C++) library for use in your own projects. It will give you some basic information about static libraries and dependencies that will be the foundation for a series of posts regarding working with third-party libraries.

## Introduction

Many developers nowadays know too little about how to work with compiling libraries. And that’s a real shame since they are an essential tool in today’s programming landscape. Virtually all programs you can write nowadays use dozens of them. Even “Hello World”.

Maybe the reason people feel intimidated by C libraries is that they involve some general UNIX knowledge. Most of the things I personally know about them is from playing around with Linux. By doing so, I learned a couple of things about dependencies and how libraries work together by separating responsibility for different aspects of functionality.

Another difficulty comes from the fact that there are different types of libraries - static libraries, shared libraries, frameworks... While they aren’t that different in the way they’re used by developers, it’s yet another thing we need to keep in order. I’m not going to get into much detail about that, so if you’re interested, you way refer to [this article explaining the difference between library types](http://www.tldp.org/HOWTO/Program-Library-HOWTO/static-libraries.html).

## A quick word on dependencies

You might not be familiar with the concept of dependencies. But I’m pretty sure most, if not all, of your project have dependencies. For example, your iOS applications depend on the Foundation framework and UIKit (and possibly many others). Foundation and UIKit, in turn, depend on tons of other frameworks and libraries. In order for your app to properly run, all these dependencies have to be satisfied. This means that if Xcode doesn’t have any of these libraries, your project wont run.

The same way you can use a library to use functionality you don’t want to write yourself, libraries can depend on other libraries. That’s how you get dependencies in “real life”.

Let make an experiment and see what dependencies some programs have. In OSX, you can easily do this with the following command:

```bash
    $ otool -L <file>
```

It will give you the names and versions of all libraries used by the executable. For instance, here’s what the ls command (that lists folder contents and file information) uses:

```bash
    $ otool -L /bin/ls
        /bin/ls:
	    /usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
	    /usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version     
        5.4.0)
	    /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version
        1213.0.0)
```

Really neat. But how about something from Cocoa. Here’s the output from a sample console utility I wrote recently:

```bash
$ otool -L justextutil
justextutil:
	/usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 120.0.0)
	/usr/local/lib/libpcreposix.0.dylib (compatibility version 1.0.0, current version
    1.3.0)
	/System/Library/Frameworks/Foundation.framework/Versions/C/Foundation
    (compatibility version 300.0.0, current version 1151.16.0)
	/usr/lib/libobjc.A.dylib (compatibility version 1.0.0, current version 228.0.0)
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version
    1213.0.0)
	/System/Library/Frameworks/CoreFoundation.framework/Versions/A/CoreFoundation
    (compatibility version 150.0.0, current version 1151.16.0)
```

Hey, we can see some familiar faces here like CoreFoundation! Oh man, otool is the best!

But seriously... Dependencies will make you cry sometimes. They will cause “undefined symbol” linker errors that will make you question your intelligence as well as sanity. Consider yourself warned. So get to know (and love) them!

## Anatomy of a library

So what is a library exactly? How do you recognize it when you see it?

A library is a collection of object files, combined in an archive that is not compressed.

Huh?! Well, you might think of it as a bundle of functions and classes all combined in a single file. Possibly having the “.a”, “.so” or “dylib” extension. It gets linked alongside your program and whenever your code calls a method from the library, the linker looks up the position of that code within the library file and assembles the resulting binary with it. It is important to note that functionality inside the library file are already compiled. That’s why companies love them so much. They can provide you with some cool functionality, but at the same time, not give you the source code for it (so you can come back for more :)).

So code from the library is automatically assembled into your binary by the linker. But what about compiling? How do you compile function calls that you don’t have the source code for. If you thought of that while reading the previous paragraph, treat yourself to a pat on the back.

In reality, the library file is not the only thing you need. You also need the library’s header files. Or at least the public ones. You add them to your project so that the compiler knows that this functionality you are using, really exists (actually it is functionality you **promise** that exists). To summarize, in order for you to successfully use third-party libraries in your project, you need two things - the library itself (most often a “.a” file) and the library’s public header files [1](#public_headers_note).

Hm, well that’s inconvenient. Why should there be two sets of files? Wouldn’t it be great if there was a single item containing them all?

Yes! Yes, there is? It’s called a framework. A framework is a library, bundled together with it’s headers and resources (among other things). You can even see this in Xcode. In the navigator (the file tree on the left), find a framework, click on the chevron on it’s left to expand it. It shows a “Headers” directory. Open that and you’ll see all header files available for this framework.

![Framework contents]({{ site.url }}/images/2015/02/frameworkContents.png "Picture showing the Foundation framework's tree structure as shown in Xcode")

## Lets start compiling libraries!

Lets start with something I had to deal with recently. We are going to be compiling PCRE for our OSX machines. PCRE, or Perl Compliant Regular Expressions is a C++ library that adds regex support. It is used by some major software projects like Apache and Apple’s Safari and if you someday need to use another C/C++ library there’s a chance that it depends on PCRE (exactly what happened to me last week).

Right, so since we’re going to be building stuff, first of all, we are going to need some source code. Conveniently enough, the [source code for PCRE](http://pcre.org) is readily available online. Head over there and download a copy of it. Be sure to get PCRE and not PCRE2 since we’re old school and don’t want any goodies from these newer versions (though I doubt PCRE2 will have different build method). So, you should be looking at an archive, containing the sources. Go ahead and unarchive it either with your GUI tool, or if you’re really hardcore, from the Terminal using:

```bash
    tar xzvf filename.tar.gz    # for tar.gz
    tar xjvf filename.tar.bz    # for tar.bz
```

It’s always good to be able to do such basic stuff like that right from the Terminal so that you can impress the cool kids, whenever they are around.

Anyway, you should end up with a directory full of wonderful stuff. Alongside the actual source code, we are going to be focusing on several files:
* autogen.sh
* configure
* MakeFile

### Autogen

If you check now, you’ll see that the configure file is not there. Autogen is another script that will generate it for you. Note that many libraries don’t have it. In this case, just ignore this step and go straight to configure.

### Configure

Configure is a shell script that contains a lot of cryptic code I cannot even begin to understand. But it will prepare your source code for build. Go ahead and run it without any parameters:

```bash
    ./configure
```

You can add additional parameters in order to build the library in a different way. For instance, if you want to build it for another architecture, like arm or arm64. Quite useful for iOS developers. However, this is outside the scope of this article and we are going to tackle it separately. So stay tuned to this blog to find out. You can always subscribe in order to get notified about new posts.

### make

Once configure finishes successfully, we can go ahead and run make. But before that, make sure configure didn’t fail! This happens sometimes. Read the output carefully and make sure there are no errors. If everything is fine, run make. It will build the library. If it completes successfully, there should already be a built version of the library somewhere inside the directory, most likely in the “.lib” directory. If that’s ok with you, you can stop here, otherwise continue with the next step.

### make install

Running `make install` will install your library in the appropriate places, like `/usr/local/lib` for instance. You might have to run in as a super user with `sudo make install`.

So, to summarize, the following sequence of commands should be enough to build the PCRE library:

```bash
./configure
make
make install
```

Now that you have the compiled library file, you are ready to use it in your projects. You will also need the public header files, but you can get those from the source code directory. Additionally, `make install` should copy them in the directories Xcode searches for library headers (e.g. `/usr/local/include/`).

Next, we may need to setup our Xcode project so that it knows where to look for our library and it’s headers. But that’s a problem for another post. I don’t want to fill your brains with too much information at once.

So that’s it for today. Hope I didn’t scare you away from compiling libraries. And don’t worry - it might take a certain amount of fiddling, but it works out fine eventually.

In the next articles, we are going to be looking at:
* Setting library and header search paths
* Compiling libraries for different architectures

You may want to subscribe to the blog to get notified for these new posts.

<a name="public_headers_note"></a>

[1] What makes a header public is purely the developers choice. A library author may not want to expose all library functionality to it’s users. He/She might choose to leave some headers hidden. Thus, the rest of the headers are the public ones.
