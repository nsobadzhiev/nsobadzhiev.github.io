---
id: 212
title: UISplitViewController tutorial
date: 2014-05-22T19:22:44+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=212
permalink: /tutorials/uisplitviewcontroller-tutorial/
categories:
  - Tutorials
tags:
  - ios
  - ObjC
  - Objective-C
  - UISplitViewController
---
# [UISplitViewController][Apple docs on UISplitViewController]

Nowadays, UISplitViewController is a de facto standard in the iPad's user experience scene. It looks great, it feels great and it also performs great. Whenever you have to show information devided into sections or have a hierarchy that drills down going into more and more detail, UISplitViewController is there for you. On the developer's side, it is mostly straight-forward to use and easy to setup as longer as you take some considerations into account.

UISplitViewController is a class from UIKit available exclusively to iPad users. It has the ability to show two view controllers on screen at the same time. The first one, called master, is located on the left-hand side of the display and the second one, the detail, takes the rest of the screen to the right. It allows developers to take advantage of the iPad's larger dimentions in order to show more information to the user at once. If you (for some strange reason) don't recall what a UISplitViewController looks like, just look at the iPad's Settings application:


![Settings Split View Controller]({{ site.url }}/images/2014/05/settings_split_view_controller.png "UISplitViewController in the Settings app")

# Creating a UISplitViewController

Creating a UISplitViewController is not what you call hard. Actually, you just need to instantiate one and set its `viewControllers` property, like so:

```objc
MasterViewController *masterViewController = [[[MasterViewController alloc] init] autorelease];
UINavigationController *masterNavigationController = [[[UINavigationController alloc] initWithRootViewController:masterViewController] autorelease];

DetailViewController *detailViewController = [[[DetailViewController alloc] init] autorelease];
UINavigationController *detailNavigationController = [[[UINavigationController alloc] initWithRootViewController:detailViewController] autorelease];

UISplitViewController* splitViewController = [[[UISplitViewController alloc] init] autorelease];
splitViewController.delegate = self;
splitViewController.viewControllers = @[masterNavigationController, detailNavigationController];
```

Not so bad, is it? Nevertheless, let's inspect what this code actually does.
First of all, the two view controllers, the master and the detail, are created. Note that in this example they are pushed into navigation controllers, but that's not mandatory. It just allows us to have a stack of views on each component and also show a navigation bar on the top of both of them. If that's not something you are interested in, feel free to remove it.

After that, the actual UISplitViewController is initialized. As promised, the `viewController` property is set. It expects an array containing **exactly** two view controller objects. The first one will be the master and the second - the detail.

Aaaand, that's it!

# [UISplitViewControllerDelegate][Apple docs on UISplitViewControllerDelegate]

In the example above, you might have noticed that we are setting `self` as the UISplitViewController's delegate. Each delegate should conform to the UISplitViewControllerDelegate protocol allowing it to get notified about changes in the controller's layout. These are mainly notifications about user interface orientation changes. By default, UISplitViewController deals with them by hiding the master view controller in the two portrait orientations. This behavior can be altered via the

```objc
- (BOOL)splitViewController:(UISplitViewController *)svc shouldHideViewController:(UIViewController *)vc inOrientation:(UIInterfaceOrientation)orientation
```  

method. Also, even if your left-hand side view is hidden, you can make sure users can access it using a swipe gesture and/or a bar button.
In order to use the former, you need to make sure the `presentsWithGesture` property of your UISplitViewController is set to `YES`.

**Note:** `presentsWithGesture` has a default value of YES so you might as well not set it at all.

Once that's done, you should be able to just swipe your finger from the edge of the screen towards the middle in order to make the master view controller magically appear.

Swipe gestures can be a nice gimmick (a gimmick is a unique or quirky special feature that makes something "stand out"), but I honestly doubt many users know that they can use the UISplitViewController like that. So it can prove handy to also have a designated button for expanding the hidden view. Luckily enough, this can be done easily using UISplitViewControllerDelegate. All you need to do is implement the following methods:

```objc
- (void)splitViewController:(UISplitViewController *)splitController
     willHideViewController:(UIViewController *)viewController
          withBarButtonItem:(UIBarButtonItem *)barButtonItem
       forPopoverController:(UIPopoverController *)popoverController;

- (void)splitViewController:(UISplitViewController *)splitController
	 willShowViewController:(UIViewController *)viewController
  invalidatingBarButtonItem:(UIBarButtonItem *)barButtonItem;
```

They allow you to have a UIBarButtonItem that your users can use in order to expand and collapse the master controller. Here's a sample implementation of these methods:

```objc
- (void)splitViewController:(UISplitViewController *)splitController
     willHideViewController:(UIViewController *)viewController
          withBarButtonItem:(UIBarButtonItem *)barButtonItem
       forPopoverController:(UIPopoverController *)popoverController
{
    barButtonItem.title = NSLocalizedString(@"Master", @"Master");
    [self.navigationItem setLeftBarButtonItem:barButtonItem
                                     animated:YES];
    self.masterPopoverController = popoverController;
}

- (void)splitViewController:(UISplitViewController *)splitController
     willShowViewController:(UIViewController *)viewController
  invalidatingBarButtonItem:(UIBarButtonItem *)barButtonItem
{
    // Called when the view is shown again in the split view, invalidating the button and popover controller.
    [self.navigationItem setLeftBarButtonItem:nil animated:YES];
    self.masterPopoverController = nil;
}
```

As you can see, they don't do much apart from making sure the button has the right title and is added to the navigation controller. Here's why the first code snippet in this article had both the master and detail controllers in navigations stacks (UINavigationViewController) - we needed it to support this bar item. Additionally, having a navigation bar makes our screen have a more traditional UISplitViewController appearance.

# UISplitViewController demo

Of course, no tutorial is complete without a demonstration of some sort. And for good reason too! After all, surely I have failed to answer at least a few of your UISplitViewController questions. So I have compiled a sample application just for you. It shows a traditional master - detail view controller. On the left, you will be able to see a list of several popular office file formats and if you select one, a bigger image of it will display on the right. It supports different interface orientations, a swipe gesture and a bar button to hide the left-hand side view. You can enjoy a screenshot of it here:

![UISplitViewController Demo]({{ site.url }}/images/2014/05/uisplitviewcontroller_demo.png "A screenshot of the UISplitViewController demo app. It shows a master view controller with the thumbnail for Pages selected, and a bigger image of the Pages logo in the detail controller")

Beautiful, isn't it? And useful too!

Anyway, without further ado... the [UISplitViewControllerDemo project][UISplitViewControllerDemo link].

Congratulations! You've successfully finished the UISplitViewController tutorial. Go tell your friends and family about it! It was, as always, a pleasure for me to introduce the topic to you guys (whoever you are). Until next time!

Thanks for reading!

[Apple docs on UISplitViewController]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UISplitViewController_class/Reference/Reference.html
[Apple docs on UISplitViewControllerDelegate]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UISplitViewControllerDelegate_protocol/Reference/Reference.html#//apple_ref/occ/intf/UISplitViewControllerDelegate
[UISplitViewControllerDemo link]: {{ site.url }}/images/2014/05/UISplitViewControllerDemo.zip
