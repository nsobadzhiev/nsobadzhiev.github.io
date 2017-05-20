---
id: 317
title: WatchKit tutorial
date: 2014-11-26T11:59:15+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=317
permalink: /tutorials/watchkit-tutorial/
categories:
  - Tutorials
tags:
  - ios
  - iWatch
  - Swift
  - WatchKit
---
![WatchKit logo]({{ site.url }}/images/2014/11/watchkit_2x.png "WatchKit logo")

# Do you even iWatch, bruh?

Sorry, I just had to write that. Anyway, we developers have to stay up to date with new technologies. And which is the newest gadget out there in the iOS scene (no, it's not beacons). It's the iWatch. I, personally, wouldn't buy it any time soon (or any kind of smart watch for that matter). I'm doing just fine with my Citizen "dumb" watch, thank you. But that doesn't mean the iWatch doesn't interest me in a professional point of view. And since Apple recently published some developer resources about it, I thought I should write this WatchKit tutorial.

# The WatchKit

So I watched the video... I read the developer guide... and I must say, I'm not that impressed. Guess I'm a romantic when it comes to new tech. Truth is, the iWatch is so tightly coupled with your iPhone that it barely serves any purpose at all, apart from displaying notifications. Even in the iOS simulator, it is listed next to the External monitor settings :).

![iWatch in the iOS simulator]({{ site.url }}/images/2014/11/iWatchOptionsSimulator.png "Xcode screenshot showing that iWatch is accessible through the iOS Simulator's external monitors section")

But all things considered, a smart watch **is** just a way to see some information without getting your phone out of your pocket so I really can't complain.

All that can be clearly seen by looking at the WatchKit framework. Almost everything is run by your iPhone, leaving only a Storyboard to the watch. Even `IBActions` are handled on the phone.

![WatchKit architecture]({{ site.url }}/images/2014/11/WatchKitArch.png "A diagram showing flow of data between an iPhone and iWatch. WatchKit joins the Watch app with the WatchKit extension running on the phone.")

From the diagram above, you can see that an iWatch app goes nowhere without it's big brother, the WatchKit extension. It only presents a predefined storyboard while the iPhone handles all "heavy-lifting". And when I say "predefined" I mean that you cannot even dynamically create UI elements. The best you can do is put all elements in the storyboard and have some of them hidden. Not good...

Anyway, since this is a WatchKit tutorial, I'll stop with the whining.

# The WatchKit tutorial

Right, first of all, a WatchKit tutorial should start with installing and enabling iWatch developement.

## Install Xcode 6.2 beta

After a while, WatchKit will be available in Xcode's stable version and most of the people reading this will not need to go through that step but for now, you need to download and install the latest Xcode beta. You can check what version of the IDE you are running and if it is 6.2 or above, you should be all set.

All the rest of you, head over to [Apple's developer portal][Apple dev member center] and download the latest beta. For those who do not participate in Apple's iOS developement program, I'll have to stop you right there. You will no have access to beta versions. Sorry guys... maybe after iOS 8.2 comes out. To summarize:

**Note:** You will not be able to download Xcode beta without participating in Apple's iOS development program.

## Getting started with the WatchKit tutorial

After installing Xcode 6.2, the first thing I did was try to create a new project, using iWatches. But, there was no such thing...
As discussed above, WatchKit involves running code on the phone. You can only add WatchKit support to existing projects. So, to get started with this WatchKit tutorial,
go to "Add Target" and select "Apple watch". This will setup everything you need in order to start using WatchKit for this project. You can also do it manually, but since it creates new targets and storyboards, it is easier to just use the wizard.

## New terminology

While reading about WatchKit, you will probably hear a few terms that you are unfamiliar with. Lets see what they are:

### Glances

The idea behind Glances is actually quite simple. The iWatch is designed so that users can quickly access information about the apps in their phones. This is achieved by allowing each app to have a **single**, nonscrollable screen that summarizes the current state an application is in. This special view, or more specifically scene in your storyboard, is called a Glance. It's important to note that Glances are non-interactive and they shouldn't contain controls, such as buttons and sliders. If a user taps on a Glance, the corresponding Watch app is launched.

### Custom Notification interfaces

![Custom notification interfaces]({{ site.url }}/images/2014/11/iWatchNotifications.png "Picture showing iWatches with custom notifications on screen")

In the (quite recent) past, local and remote notifications were static and managed by iOS. When iOS 8 came along, they became interactive. For the first time, developers were able to do some minor customization on what their notifications did. WatchKit takes this even further by allowing apps to provide custom appearance and functionality to the notifications displayed on the iWatch. They still cannot be very interactive, but they can look different than the generic ones. Tapping on a custom notification results in launching your Watch app, so using buttons and other interactive controls is unavailable. There are two kinds of notifications interfaces:

#### Static notification interfaces

The static notification interface is a simplified version of your notification appearance. It can only contain static images and text. In fact, the only item that changes is the notification text. This is done by connecting the `notificationAlertLabel` outlet to a WatchKit label.
The idea behind this is that if the iWatch fails to initialize your "real" notification in time, it will display this static one. I'm not sure how often this will be, but either way, you are required to support that.

#### Dynamic notification interfaces

Dynamic notification interfaces is what we were waiting for. It allows you to specify a custom "view controller" to display the notification. And when I say "view controller", I don't mean UIViewController unfortunately. It's WKUserNotificationInterfaceController, a WKInterfaceController subclass - UIViewController's crippled brother. In fact, WatchKit doesn't use UIControls anywhere. It has it's own (rather sad) UI elements. But more on that later.

Dynamic notifications in WatchKit are not interactive (apart from the action buttons that you can add). They should be designed to only display information. Tapping on the notification will result in opening the Watch app.

In your WKUserNotificationInterfaceController subclass, you will rely on the following methods in order to handle incoming notificaitons:



Swift

	func didReceiveRemoteNotification(_ remoteNotification: [NSObject : AnyObject]!, withCompletion completionHandler: ((WKUserNotificationInterfaceType) -> Void)!)

	func didReceiveLocalNotification(_ localNotification: UILocalNotification!, withCompletion completionHandler: ((WKUserNotificationInterfaceType) -> Void)!)

Objective-C

	- (void)didReceiveRemoteNotification:(NSDictionary *)remoteNotification
                      withCompletion:(void (^)(WKUserNotificationInterfaceType interface))completionHandler

	- (void)didReceiveLocalNotification:(UILocalNotification *)localNotification
                     withCompletion:(void (^)(WKUserNotificationInterfaceType interface))completionHandler

### Glances and notifications in the storyboard

Everything in an iWatch app is done via the Storyboard. Everything! So how do you add your glance and notifications controllers in there. Well, every storyboard has an entry point right? WatchKit introduces three new ones. A typical Storyboard for a Watch app, contains a Main, a Glance and a Notification entry point. If you go through the "Add new Apple Watch target" wizard, these are already setup for you, apart from the Glance. To add the latter manually, just add a new "Glance interface controller" to the scene and Xcode creates it for you.  

## WatchKit tutorial - the framework

Let's finally talk about the classes in the WatchKit framework. Truth is, there aren't that many. I was a bit disappointed. A whole new devices is just managed by a handful of classes. The main ones are:

### WKInterfaceController

WKInterfaceController in WatchKit is the same as UIViewController in UIKit. I'm a little disappointed that we cannot use our standard Cocoa classes, though.
Anyway, WKInterfaceController is a lot more restricted and has a different life cycle. Instead of methods like `viewDidLoad`, `loadView` and `ViewWillAppear`, there are:

#### initWithContext:

This is the designated initializer for WKInterfaceController. The context parameter is used in order to pass any data from a previous interface controller to this one. Is it just me or there is a little influence from Android here?

#### willActivate and didDeactivate

These are the methods you'd override in order to get notified whenever an interface controller becomes visible or is being dismissed. `willActivate` is the place you will need to make sure your controller's information is up to date and you can use `didDeactivate` in order to clean up.

### WKUserNotificationInterfaceController

WKUserNotificationInterfaceController is a WKInterfaceController subclass that is used to display custom notification interfaces (more information about that above).

### WKInterfaceDevice

This class contains same basic information related to the iWatch. This includes screen bounds, scale and current locale. You can also use it to manage your images cache. WatchKit allows you to store up to 20 MB of cached images on the iWatch itself in order to reduce latency time whenever they are needed. Remember that the iPhone and iWatch communicate via bluetooth - transferring images can take a while.

### WKInterfaceObject

This is the base class for all UI controls supported by WatchKit. As I said, you cannot use UI views from UIKit. All you can do is use the ones that WatchKit provices for you. These are:

* WKInterfaceButton
* WKInterfaceDate
* WKInterfaceGroup
* WKInterfaceImage
* WKInterfaceLabel
* WKInterfaceMap
* WKInterfaceSeparator
* WKInterfaceSlider
* WKInterfaceSwitch
* WKInterfaceTable
* WKInterfaceTimer

### WatchKit layout

WatchKit does not use Autolayout to position it's element. This is a bit weird, bit I guess it was too complex for the slow iWatch. So Apple came up with a layout system that positions elements relative to each other in a horizontal or vertical order.

_LinearLayout! Again, some Android deja-vu..._

All this is controlled in Interface builder via the Attributes inspector. And for something this simplistic, it works rather well.

![Attributes inspector]({{ site.url }}/images/2014/11/WatchKitAppearanceOptions.png "Picture showing appearance options for WatchKit controls")

## WatchKit tutorial - the code

Finally, the code!. I've created a sample project where you can see how everything is setup for WatchKit developement. It doesn't do much, but it should be enough to get you started with WatchKit. You can [download it here][WatchKit tutorial source]. Here it is in action:

![WatchKit tutorial sumulator run]({{ site.url }}/images/2014/11/WatchKitSimulatorRun.png "Picture showing the iOS simulator running the sample application")

If this WatchKit turorial was enough to get you excited about iWatch developement, you can to [Apple's WatchKit programming guide][Apple docs on WatchKit] and learn more.

[Apple dev member center]: https://developer.apple.com
[WatchKit tutorial source]: {{ site.url }}/images/2014/11/WatchKitTutorial.zip
[Apple docs on WatchKit]: https://developer.apple.com/watchkit/
