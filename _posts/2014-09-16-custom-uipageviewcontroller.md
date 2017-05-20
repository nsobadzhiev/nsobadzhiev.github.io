---
id: 285
title: Custom UIPageViewController
date: 2014-09-16T16:04:41+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=285
permalink: /tutorials/custom-uipageviewcontroller/
categories:
  - Swift
  - Tutorials
tags:
  - DynamicPageViewController
  - ios
  - Swift
  - UIPageViewController
  - UIScrollView
---
# The traditional UIPageViewController

A few months ago, I wrote an [article about using UIPageViewController][UIPageViewController tutorial]. I even wrote about dynamically exchanging view controllers inside of it. However, I am a bit embarraced to admit that, even though the code I used there works, I don't actually use it for my projects. Shortly after I posted that tutorial, I realized that it is too problematic and decided to write my own custom UIPageViewController. Ever since, I've felt I needed to post another tutorial to let my readers know that there is a better way to dynamically change pages.

Anyway, if you feel that you still want to try the conventional UIPageViewController (which is perfectly fine in most cases), you might want to go read my [UIPageViewController tutorial article][UIPageViewController tutorial].

# Why should you use a custom UIPageViewController

As I said, using a UIPageViewController is perfectly fine in most cases. In fact, It's a fantastic class that is both excellent for user experience and easy to use. However, since it relies on a data source to provide it's "pages" and it also enforces a caching policy to ensure smooth scrolling and avoid unnecessary creation of view controllers, it can get a bit hard to dynamically insert and remove pages. In the [UIPageViewController turotial article][UIPageViewController tutorial] I wrote how you could force it to clear it's cache, allowing you to exchange controllers, but I found that even then there are problems (and by problems I mean crashes).

It was then that I realized that it was nothing more than a UIScrollView with paging enabled and having each page as a child view controller. It shouldn't be that difficult to write a custom UIPageViewController. So I did...

# DynamicPageViewController - a custom UIPageViewController

DynamicPageViewController is a class that mimics the bahavior of UIPageViewController, but also allows you to insert and remove pages on demand (even if you want to remove the currently displayed one). It relies on an UIScrollView to layout it's views and support a pan gesture to turn pages.

It's interface is rather simple. It maintains a `viewControllers` property that holds all view controllers that represent pages in the custom UIPageController. If you modify that array, the views will automatically be redrawn to refrect the change.

Also, you can use the `insertPage` and `removePage` methods in order to change the layout.

	func insertPage(viewController: UIViewController, atIndex index: Int)
	func removePage(viewController: UIViewController)

To summarize, these are the only items in DynamicPageViewController's interface you would generally care about:

* `viewControllers` property _(maintains an array of currently displayed pages)_
* `insertPage` method
* `removePage` method

Without further ado... the custom UIPageViewController - DynamicPageViewController. In order to be in line with the latest fashion, it's written in Swift, but it shouldn't be that difficult to port to Objective-C, if that's what you need. it is released with an MIT license and is freely available for download in [github][DynamicPageViewController repo].
Enjoy!

_**NOTE:** An MIT license means that you will be able to use it without restriction in your projects (free or commercial) without disclosing who wrote it_
	

[UIPageViewController tutorial]: http://devmonologue.com/ios/tutorials/uipageviewcontroller-tutorial/
[DynamicPageViewController repo]: https://github.com/nsobadzhiev/DynamicPageViewController