---
id: 395
title: Snapshot testing with FBSnapshotTestCase
date: 2015-11-02T14:03:32+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=395
permalink: /ios/snapshot-testing-with-fbsnapshottestcase/
categories:
  - iOS
  - Test Driven Development
  - Testing
tags:
  - FBSnapshotTestCase
  - ios
  - Objective-C
  - snapshot testing
  - TDD
  - unit testing
---
If you had asked me two weeks ago about snapshot testing, I'd definitely tell you it's stupid. And I think I had my reasons... I don't regret it. But I guess today's opposite day, because my opinion turned 180 degrees. I haven't already made my mind up, but it seems like snapshot testing is quite promising.

## What is snapshot testing

I'm sure you've already heard about it, probably from your hipster friend. Basically, snapshot testing is a way to automatically test the aesthetic part of your UI. It consists of creating a view from your app and comparing it against a reference image. It they match, all is good and people are happy. But if even one pixel is off, women scream and children cry.

_I think that's exactly what designers expect to happen when something is one pixel off._

## Man, snapshot testing is lame

Just a few sentences in and many of you probably already scream at the monitor and have foam forming in the corner of your mouths. And I know what you're thinking.

1. How can the screen be exactly the same as the picture?
2. How can I generate an image that's exactly identical to my UI?
3. How can I make these tests repeatable with a pixel-to-pixel precision?
4. Isn't comparing images totally slow?
5. How do I know what exactly is different between the two images?
6. How do I live with myself knowing my test fail for no reason... which they will?
7. When I change my UI's appearance tomorrow, who's going to give me new reference images? Not me!

Of course, these are all good points. I feel the same way :). But let me show you how one little framework attempts to answer all those quiestions.

## Introducing [FBSnapshotTestCase][Link to github page]

FBSnapshotTestCase is a framework developed by a company you might have heard of. It's called Facebook.

> Maaaan, snapshot testing... Facebook... This article sucks!

It probably does... but not because of FBSnapshotTestCase.

Aaaaaanyway, what the FBSnapshotTestCase library does is not a lot, but it's enough.

### An XCTestCase subclass

The first thing that deserves credit is the fact that the FBSnapshotTestCase class inherits from XCTestCase, essentially making comparing snapshots a unit test. And yes, I know this totally violates many of the core principals of unit testing and TDD, but still it's useful to get all the goodies from it. So what kind of goodies are we talking about:

#### Xcode integration

To Xcode, your FBSnapshotTestCase suite is just another unit test collection. So you get all the IDE support for it like:

* CMD+U to run tests
* UI for showing test results
* Easy continuous integration

#### Reliability

Since you are not running your app with real data, you can isolate different screens and even views. That makes it easy to test different components in isolation using hardcoded data. This significantly reduces the possibility of random failures. Go Facebook!

#### Isolation

I'm repeating myself a little here, but I cannot stress enough how useful is to test components in isolation. You are not limited to testing the whole screen. Just instantiate a view and see if it looks like it should. Allocate a view controller, inflate a Xib, load a storyboard. It's all on the table.

### Recording mode

One question I've always asked myself when thinking about snapshot testing is... **Who the hell creates all those reference images and how come they match the app 100%?**. The answer is recording mode...

FBSnapshotTestCase lets the app generate reference images for itself. The class has a property named `recordingMode`. If set to YES, instead of comparing images, it will make a snapshot of your view and store in the in the reference images folder. Essentially, whenever you've verified that your view looks good, you can run your test in recording mode and it will become a reference for all subsequent runs.

Well, it still sucks that you need to change some property in order to generate reference image. And then, you will probably forget to remove the recording mode and suddenly tests start failing... But anyway, life's not perfect.

### Playing nice with animations?

So what if you've done some animations that go along with your views - a snapshot will only capture the initial position.

Not necessarily. Most of the time, animations are fine. Since they mostly work on layers, UIView's `drawViewHierarchyInRect` method is smart enough to render the view's real... uhm... final position. Thus most of your animations are safe and you will be able to get an image of your UI **after** they are done.

#### performSelector: ?

Oh, you got me there... If your views rely on NSObject's `performSelector` for delaying actions, you might (and will) run into problems with FBSnapshotTestCase. Well, theoretically you **can** try to work around it...

#### Runloops?

For the above mentioned `performSelector` and other time-based issues you might try the following:
```objc
[[NSRunLoop currentRunLoop] runUntilDate:[NSDate dateWithTimeIntervalSinceNow:1.0f]];
```
This will effectively activate the runloop for 1 second and force it to execute everything scheduled on it. However, I wouldn't recommend using it unless you have no other choice. And let's be honest... you always have another choice :).

The problem with running the runloop is that it will autimatically increase your tests in length. For the above example, that's one extra second. And it's a secred TDD rule to have tests that are fast... really, really fast. The idea behind is that developers should be running test all the time during implementation. And if tests take a minute to complete, they're are just not going to do that.

Even though FBSnapshotTestCase is not exactly unit testing and running snapshot tests every 2 minutes during development is not that useful, it still helps to follow TDD rules and keep execution times to a minimum. _Spoiler alert - 5-10 seconds should be more than enough for all your FBSnapshotTestCase suits_.

### Changing code to make it more... testable

As you can see, most of your UI classes can be snapshot tested without modification. But if some of them fall victom to the symptoms we talked about in the last chapter, you will be faced with a decision:

1. Change your production code to allow snapshot testing
2. Skip testing for that particular item

The second option might be tempting, and depending on how important and risky that component is, it may be the best solution. But still, the first option is worth a try!

#### To change production... or not to change production

It's an old philosophical question... Should you change production code in order to make it more testable, or should test code try to adapt to production code. Some developers agree to the former, some to the latter. Test-driven development for one thing has the core value of shaping your production code according to your tests. On the other hand, many programmers would argue that if a testing platform forces you to change your application, it's obviously not doing its job.

So the truth is out there (_X-files anyone?_). If you are dumb enough to ask me, the answer depends on some additional factors.
First of all, I agree that the testing "harness" shouldn't interfere with production code. That being said... there's an exception! So:

The testing "harness" shouldn't interfere with production code... **unless it forces you to change it for the better!**

And that's exactly where TDD comes in. Yes, it forces you to change the way you write production code, BUT only to make you write better structured programs.

I like to consider non-testable or hard to test classes as [code smells][Code smell definition]. If you find it hard to put into a test harness, it's probably because it's too tightly coupled to something or it tries to be more than it should ([Single responsibility principle][Single responsibility principle definition] violation). So it might be worth fixing that, instead of trying to make your tests work around it.

Granted, this holds true more about TDD and less about Snapshot testing with FBSnapshotTestCase. If you're not able to create the reference image you want, that doesn't mean your class is not well structured.

#### Coping with unwieldy animations

This is going to sound terribly underwhelming but a universal way of doing that is to... not have animations in the first place. I would like to bring a common pattern from UIKit to your attention. For example, the method for changing the value of a UISwitch:
```objc
- (void)setOn:(BOOL)on animated:(BOOL)animated;
```
You can specify if you want the animation or not. Normally, the purpose of this is that if you want to change the value while your view is not on screen and the user is not going to see it, you might choose to have it not animated.

So being able to turn off animations can be a cool thing to have. Even if you don't use it right for now. (And yes, I do realize writing code you're not using violates some core principals of TDD, but hey, I also like to live life dangerously :) ).

## Adding FBSnapshotTestCase to a project

### Get the framework

There are two ways to add FBSnapshotTestCase to your project. The easy way... and the way I did it. The easy way is CocoaPods. Yes, FBSnapshotTestCase supports CocoaPods so chill. So if you're one of those guys that use CocoaPods, you're all set. But the thing is, I don't love myself enough to use it. What I did was to compile FBSnapshotTestCase myself. In fact, that was kind of simple. The folks in Facebook already have a target in their Xcode project that generates a .framework file. So all I did was build it. The trick is that you might want to compile for multiple architectures. But even that is easy. If you are nerdy enough to be interested in that, you can read my [Compiling libraries article][link to compiling libraries article].

### Reference directory

One thing you need to do in order to setup FBSnapshotTestCase correctly is to set the `FB_REFERENCE_IMAGE_DIR` variable. Again, there are a couple of options. The lazy way is to set it directly in code somewhere. But lets face it - that's not how the cool kids do it. Instead (and this is what the authors recommend) they would set it as an environment variable. As most of you probably already know, that's done in the "Edit Scheme screen".

![Scheme settings image]({{ site.url }}/images/2015/11/FBSnapshotTestCase_EnvVariable.png)

One important note here is that you need to make sure the " alt="" /> @implementation LoginSnapshotTests
```objc
- (void)setUp {
    [super setUp];
    self.recordMode = NO;
}

- (void)tearDown {
    // Put teardown code here. This method is called after the invocation of each test method in the class.
    [super tearDown];
}

- (void)testLogin {
    UIStoryboard* loginStoryboard = [UIStoryboard storyboardWithName:@"Login" bundle:nil];
    LoginViewController* loginController = [loginStoryboard instantiateViewControllerWithIdentifier:@"myLoginVC"];
    UIView* loginView = loginController.view;
    loginController.usernameField.text = @"dev@monologue.com";
    FBSnapshotVerifyView(loginView, nil);
}
```
I hope that the `setUp` and `tearDown` methods need no introduction? They are run before and after each test in order to prepare the environment for the test. In OCUnit, all methods returning `void` and starting with the word `test` are considered tests, hence the name `testLogin`. What follows is a function body so simple even the most hardcode TDD fans wouldn't complain (well, maybe we can move the UIStoryboard stuff in the setUp method).

As discussed above, you can use `self.recordMode = (NO|YES);` to toggle between generating reference images and actual testing.

First of all, we are opening the storyboard to create an instance of the login view controller. Nothing interesting there, but what happens afterward is really important. We are (somewhat artificially) calling the getter for the view controller's view property. This will force the instance to call the famous `loadView` and `viewDidLoad` methods to fully initialize its UI elements. If you fail to do so... well, lets just say you'd be disappointed with your reference images.

With that out of the way, you can do whatever you like in order to make your view controller display some data. Even if you don't, I guess it would still be a valid and useful test, but I like to fully supply my UI with data in order to increase test coverage (unless I have time to make a separate test for every situation).

Lastly, we do the actual comparison. It's most of the time the same line:

```objc
FBSnapshotVerifyView(yourView, nil);
```
## Common Snapshot testing with FBSnapshotTestCase issues

### Table view cells in a storyboard

Going back to our `testLogin` method, we can conclude that what a typical snapshot test involves is:

* Creating a view using a storyboard, xib or good old fashioned code
* Making sure subviews are created, positioned and layed out
* Filling the view with contents
* Making a photo (Cheeeeeeese!)

One case where this is not as clear is with table view cells. In the port-storyboard era, many cell are created directly in their table view controller, making it unclear how to get to programatically.

But I've figured it out for you. There are several ways to get a reference to the cell you want, and some of them result in a view that's not properly layed out. So to save you a little time, here's the code that does it for me:

```objc
- (void)testStoryboardCell {
    UIStoryboard* myStoryboard = [UIStoryboard storyboardWithName:@"MyStoryboard" bundle:nil];
    SomeTableViewController* tableController = [myStoryboard instantiateViewControllerWithIdentifier:@"RandomTableViewController"];
    MyTableViewCell* cellISoDesire = [tableController.tableView cellForRowAtIndexPath [NSIndexPath indexPathForItem:0 inSection:0]];
    FBSnapshotVerifyView(cellISoDesire, nil);
}
```

So what I found to be a good way to test storyboard cells is to initialize the whole controller (I doubt you can get around that) and call `cellForRowAtIndexPath:` to make it create the cell itself.

Another option would be to make the table dequeue a cell, but that doesn't work reliably. Particularly, sometimes the layout is incorrect even if you call `layoutIfNeeded`. I assume that's a way to force the view to apply its constraint, but I haven't found it yet.

Getting back to my solution, the drawback is that the table view will start calling it's data source methods so you need to implement them in your test harness. Well, you might get around that if you are using static cells. But most of the time you'll need some test data anyway.

### Keeping reference images in sync

Creating and modifying reference images is dead simple with FBSnapshotTestCase. But you still need to remember to keep them in sync. If you change your UI even a little, you need new images. And sometimes this is a little subtle. Sometimes, you don't even know the aesthetics changed. Actually, having a test fail in a case when you change your app's appearance without knowing is a really good thing. That's what snapshot tests are for. But it can get annoying if something moves just one pixel but suddenly several tests fail. And if you rely on CI, you builds start failing.

So, apart from always remembering to run tests locally before committing I don't know what to recommend. It's hard to admit, but even after being properly setup, Snapshot testing with FBSnapshotTestCase still require some maintenance.

### Jenkins... Hudson... or whoever that guy was

A real cool thing to do with FBSnapshotTestCase is to add it to your CI jobs. Every time someone commits, the project is built and snapshot tests run. If they fail, another job starts that prints that "someone's" resignation letter and denies him/her access to the building. Oh, and also publishes a tweet that they suck. Or is it just my company that does that?

There is one problem that might occur though... Since snapshot testing with FBSnapshotTestCase is for the most part only available for the simulator (because your reference images typically reside on your Mac and the iOS device cannot access them unless you copy them into the bundle), your CI server needs to run the iOS simulator. Typically this is OK, but I personally incountered a problem with our Jenkins.

It would run the tests fine for a while but when you try again in a few hours, the job would fail, saying that the simulator failed to respond in time.
The problem is that the Simulator is a GUI application and services (daemons) have a little trouble starting those. Now, I really struggled fixing this, but in the end, I found a solution. It turns out you have to change Jenkins from being a daemon to being an [Launch Agent][Creating Launch Agents link]. The idea is that a Launch Agent has the permission to start GUI apps.

In order to [setup Jenkins as a Launch Agent you can follow this article][Jenkins as a Launch Agent article]. Basically you need to move Jenkin's plist from `/Library/LaunchDaemons/` to `/Library/LaunchAgents/`. And I suspect this is not specific to Jenkins, so you can apply it to whichever CI you're using.

## Testing on multiple devices

What I really find useful about snapshot testing is running the cases for multiple screen sizes. I don't know about you guys, but I personally mostly use a single device/simulator while developing so I always risk having layout issues on the iPhone 6+ ot 4s for example. To tackle this, I've setup a build machine task that runs my snapshot testing with FBSnapshotTestCase on several simulators.

This is actually fairly easy using either xcodebuild or xctool:
```bash
xcodebuild -project MyProject.xcodeproj -scheme "SnapshotTests" -destination name="iPhone 5s" -destination name="iPhone 6 Plus" test
```
What I'm trying to say is... Yes, you can chain destinations to execute on several devices one after another.

## Try it out, it's free!

And just like that, you've learned everything I know about snapshot testing with FBSnapshotTestCase. I hope I've convinced you it's not lame to test using images. And I hope that just like every teenage girl and every hipster, you'll start taking pictures of your UI and showing them to your frinds. You can even apply filters, I guess.

Seriously though, in the past I've always considered snapshot testing terribly unefficient and useless. But with a simple framework like FBSnapshotTestCase, it has become a breeze and by far the easiest way to automate UI testing. So stop acting prejudice and try it out!

[Test-driven development with OCUnit article]: http://devmonologue.com/ios/test-driven-development/test-driven-development-ocunit/
[link to compiling libraries article]: http://devmonologue.com/ios/using-libraries/compiling-libraries/
[Creating Launch Agents link]: https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html
[Jenkins as a Launch Agent article]: http://staxmanade.com/2015/01/setting-jenkins-up-to-run-xctool-and-xcode-simulator-tests/
[Link to github page]: https://github.com/facebook/ios-snapshot-test-case/
