---
id: 378
title: Singleton vs. single instance
date: 2015-05-20T09:28:41+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=378
permalink: /design-patterns/singleton-vs-single-instance/
categories:
  - Design patterns
tags:
  - design patterns
  - ios
  - single instance
  - Singleton
---
Today on DevMonologue we are going to be talking about a design concept that has been bothering me for a while. It is not only related to iOS development, but programming in general.

I wouldn't say I'm a expert in software architecture. But in my experience, I find that the issue of singleton vs. single instance is something many projects suffer from and many people don't realize it.

That's why I want to share my thoughts about how to avoid some major design flaws. As I said, I don't necessarily quarantee everything written here is 100% true in all cases, so all feedback is greatly appreciated. I'd love to see this article become a discussion with people sharing their experience and maybe provide solutions to common problems in this field. So don't be shy and write us a few lines in the comment section ;)

## What's that "Singleton vs. single instance" nonsense all about?

With all the "sentimental" stuff out of the way, let get to the point. What do I mean by "Singleton vs. single instance"?

### The origin of "Singleton vs. single instance"

Now, I'm not sure (historically) who introduced that term, but I can tell you where I read about it in the first place. It was in a book called "Working effectively with legacy code" by Michael Feathers. If you don't already know about that book, I advise you to check it out. There are loads of useful tips in it, and even if you consider that you're not working with legacy code, you will benefit from that book, believe me.

### The definition

The principle behind "Singleton vs. single instance" is really simple. It just specifies that wherever you're normally using singleton objects, you should consider just having a single instance. What do I mean by that?

A singleton is a class that enforces the rule that there can only be a single object of it and most often that object can be accessed throughout the program.

A "single instance" on the other hand means that the class itself is not a singleton, but (on a higher level), you just make sure there is only one instance.

At first glance, it might seem that this is not that important, and even that a singleton is better suited for the job because it is more explicit about its proper use. Lets see if I can convince you otherwise...

### The cons

#### Breaking capsulation

In both cases the class is only supposed to have a single object. But a singleton actively enforces that, whereas the "single instance" is too naive and just assumes its users will know not to create multiple objects. As we all know, it is always better to be as explicit as possible when it comes to such scenarios, so as far as capsulation is concerned, the singleton wins.

#### Ease of use

Developers are lazy and (rightfully so) like easier interfaces. And you cannot beat a singleton as far as easy access is concerned. All you have to do is import the singleton class (if you have to), call the method that returns the shared instance and you're good. It doesn't get better than that. With a "single instance" you will need to figure out who has that object and how you can get to it.

Now, ease of use is not always a good thing. And I hope a warning sign went up in your brain while you were reading the last paragraph. But more about that later.

### The pros

#### "Singleton vs. single instance" in Test-driven development

Like many of the topics you will find in the "Working effectively with legacy code" book, this one is connected to TDD. But even if you don't approve of TDD, don't close the tab just yet. Even though "Singleton vs. single instance" greatly simplifies test-driven development, it can also be a powerful design decision in all projects.

In TDD in general, it is very important that every test runs in isolation and the environment is reset between them. That makes singletons especially problematic. Since there is only one object which persists throughout the lifetime of the application, you cannot ensure what state a previous test has left the singleton in (Also note that most IDEs have the ability to run unit tests in parallel and generally don't ensure tests are run in sequence). There are ways to get around that, but generally, while using test-driven developement it is "Singleton vs. single instance" - 0:1.

#### Restricted access

This is the opposite of the "Ease of use" that we were discussing above. But why should we try to make it hard to access an object.

Well, there is an inherent problem with universal access. If all parts of an application can access an object, it is often hard to know what the problem is when something bad happens. If you track a bug to a singleton object method that 30 other objects call, it is not easy to find where the problem comes from.

Another problem is that if you make it really easy for people to use your class, they will use it... constantly. And this leads to exatly what I wrote about in the previous paragraph - everybody seems to be calling methods from the singleton class.

Now, for generic and common functionality these things are ok and shouldn't cause problems. It's not like you should never use a singleton. But I feel like people (including me) are overusing them. And I'll give you an example:

### An example of a "Singleton vs. single instance" issue

To me, the singleton design pattern is extremely useful, but many developers seem to be abusing its power. When you opt to use it for your classes, you better think really hard before you determine that's what's required. Is it __absolutely__ necessary to only have one instance... will it break your architecture if two of them exist? Or is it just that one instance is probably sufficient?

The most common abuse of the singleton pattern happens when people just need a global variable. Since global variables are (for good reason) condemned by modern programming standards, but they are still useful, developers can be tempted to create a singleton just so that an object is globally acessible. So, lets ask the question from the previous paragraph:

> Is it __absolutely__ necessary to only have one instance... or is it just that one instance is probably sufficient?

No, it isn't necessary!

However, this a rather primitive example of a poor "Singleton vs. single instance" choice. Whereas you should definitely stay away from such situations, you are unlikely to get into serious trouble with them. To get into the real problem, we will need to dig deeper.

Let consider our beloved MVC pattern. Particularly the Controller part - our business logic. How many projects have you seen or participated in, that made heavy use of singletons in their business logic. Be honest about it. CommunicationsManager, DataManager, NotificationsManager, LoginManager... these are all very likely to be singletons. But should they?

Lets think about the last one, LoginManager, for a second. It is probably an object that manages a user session - perhaps it contains a token, cookie, user credentials?
Most applications will only allow a single user to be logged in at the same time. So, there really should be only one instance of the LoginManager class, right? And the singleton pattern fits really well, doesn't it? Let ask the same question about singletons in this case:

> Is it __absolutely__ necessary to only have one instance... or is it just that one instance is probably sufficient?

Well yes, yes it is! Having two LoginManagers would be an error! And yet, something's fishy around here. What about this use case:

* Login
* Logout
* Login again with different credentials

Oh! It would seem that LoginManager should, indeed, have only one instance, but that instance might not be the same throughout the applications life. So, the question we should have been asking is actually:

> Is it __absolutely__ necessary to only have one instance that doesn't change through-out the course of the application's life?

So what happens in most projects (in my experience) is that the LoginManager instance (for example) stays the same between logins. To make that work, upon logout, all user information is deleted from the instance. It should be fairly easy. How hard can it be? A user session is only identifiable via several items - maybe a token or a username. But what about those pre-cached user friend lists... or avatar... or password. This kind of code always seems to break during maintainance. You cannot rely on your colleagues (or even you) to always remember to clear data upon logout. It just doesn't work that way.

And next time, when you forget to clear that user token... what will happen? You might even end up logging the wrong user in!

Now, if only our LoginManager was not a singleton... We would just delete the object when the user logs out and be done with it. We wouldn't be that concerned that the class might not be clearing all data during logout.

I guess the moral if this story is that similar to real life, nothing is forever in software development. So don't get too attached to your objects. Otherwise they will break your heart into small pieces!

And on that sad note, I think that's it for today's monologue about Singleton vs. single instance. Thanks for reading!