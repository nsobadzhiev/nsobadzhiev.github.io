---
id: 171
title: UIPageViewController tutorial
date: 2014-04-06T17:10:53+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=171
permalink: /tutorials/uipageviewcontroller-tutorial/
categories:
  - Tutorials
tags:
  - ios
  - ObjC
  - Objective-C
  - uipagecontroller
---
## What is a [UIPageViewController][Apple docs on UIPageViewController]?
UIPageViewController is a class from the UIKit framework that can be used in order to display a collection of UIViewControllers. Unlike other container classes like UINavigationController or UITabBarController, this class arranges all views either horizontally or vertically in a line. In order to switch between two screens, users can use the swipe gesture, use a [UIPageControl][Apple docs on UIPageControl], or any other way implemented by the programmer. It makes for an excellent user experience and can be used for displaying a collection of data in cases where it is too complicated for a table view.

You can see a prime example of using UIPageViewController in applications like [Apple's iBooks][iBooks] or even the iPhone itself - the Springboard (the main menu) works exacly like it!

![A Screenshot of the iBooks application as a page is turn, showcasing UIPageViewController in action]({{ site.url }}/images/2014/04/ibooksturn.png "UIPageViewController in action")

## [UIPageViewController][Apple docs on UIPageViewController] class overview

UIPageViewController is pretty straight-forward to use. There are only a few considerations that you need to have in mind as you use it.

### UIPageViewController layouts its child controllers in a scroll view

Though you shouldn't need to know that in order to use UIpageViewController, it is good to know that it internally uses a UIScrollView to display its data. All views are arranged side-by-side (on top of each other if you use UIPageViewControllerNavigationOrientationVertical in the navigationOrientation property). The reason I'm mentioning this is because you might end up in a situation where some of your subviews go outside the bounds of your view controller's view. In that case, these subviews might render on other pages inside the UIPageViewController.

### UIPageViewController uses a data source to retrieve its pages

Much like UITableViewController, UIPageViewController uses the data source design pattern in order to "ask" for the components it needs to render on screen. This is done via the [UIpageViewControllerDataSource][Apple docs on UIpageViewControllerDataSource] protocol. Strictly speaking you don't need to implement it in order to populate a UIPageViewController. You can pass a static set of data. The sole purpose of using a data source is to introduce _lazy loading_ to the process (lazy loading is the technique of creating objects / calculating data when it is first needed, instead of when the object is created). You will read more about that in a bit.

There are a couple of methods that you need to implement from the UIPageViewControllerDataSource protocol to get a page controller working. These are:

- (UIViewController *)pageViewController:(UIPageViewController *)pageViewController viewControllerBeforeViewController:(UIViewController *)viewController
- (UIViewController *)pageViewController:(UIPageViewController *)pageViewController viewControllerAfterViewController:(UIViewController *)viewController

As you can clearly see, the main concept behind this is to let the controller know what view you want it to display next, depending on which direction the user swipes. The first method handles going backwards and the second one forward. Unlike the table view data source, you are not given the index path of the requested view controller and that can come in handy whenever you don't necessarily know how many pages you will have (which is more often than you might think).

#### [UIPageControl][Apple docs on UIPageControl] support

As you probably know, a UIPageViewController is most often accompanied by a UIPageControl. It shows these little dots that represent which page is currently selected. Even though you can use this component outside your page controller and set it up yourself, you can also manage it through UIpageViewControllerDataSource via the methods:

- (NSInteger)presentationCountForPageViewController:(UIPageViewController *)pageViewController
- (NSInteger)presentationIndexForPageViewController:(UIPageViewController *)pageViewController

Do note that a page control will only appear for controllers with transition style `UIPageViewControllerTransitionStyleScroll`.

### UIPageViewController uses a delegate to notify about changes in its state

You can use the [UIPageViewControllerDelegate][Apple docs on UIpageViewControllerDelegate] protocol to get notified whenever your page controller makes a transition and/or the interface orientation change parameters need to be modified.

#### Page controller transitions

You need to implement the following methods:

- (void)pageViewController:(UIPageViewController *)pageViewController willTransitionToViewControllers:(NSArray *)pendingViewControllers
- (void)pageViewController:(UIPageViewController *)pageViewController didFinishAnimating:(BOOL)finished previousViewControllers:(NSArray *)previousViewControllers transitionCompleted:(BOOL)completed

They are the perfect place to update all other parts of your user interface whenever a page turns. For instance, iBooks uses them in order to go into full screen mode, assuming that if the user goes to another page, he/she is reading and wants all distractions to be removed and take advantage of the entire screen.

The first method is called before the actual transition takes place and the second one right after the animation is completed. So which one does iBooks use? If you thought "The first one", you guessed it, since the application goes into full screen even before the page is turned.

One important thing to note about these delegate methods is that they will **only be called for user initiated transitions**! This means that if you, programatically, change the page the controller is at, they will not execute. So it is critical that you make sure to setup everything you need manually in such cases.

Also, it is helpful to pay attention to the `transitionCompleted` parameter, provided by the second transition method. Its value is `YES` only if the transition is successful. A transition can be unsuccessful if the user started a swipe but never finished it, leaving him/her on the same page as before. It can be very useful to be able to read this value and determine if you really need to change anything in your application, or you can just ignore this callback.

#### User interface orientation parameters

The second part of the [UIPageViewControllerDelegate][Apple docs on UIpageViewControllerDelegate] is devoted to handling orientation. The methods are:

- (UIInterfaceOrientation)pageViewControllerPreferredInterfaceOrientationForPresentation:(UIPageViewController *)pageViewController;
- (NSUInteger)pageViewControllerSupportedInterfaceOrientations:(UIPageViewController *)pageViewController;
- (UIPageViewControllerSpineLocation)pageViewController:(UIPageViewController *)pageViewController spineLocationForInterfaceOrientation:(UIInterfaceOrientation)orientation;

The first two are used in order to specify which orientations are supported by your controller. However, the first one is a little bit special. It sets the _preferred_ orientation. If you do not know what makes an orientation the preferred one, here's what Apple's documentation says about it:

&gt;- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation
&gt;
&gt;The system calls this method when presenting the view controller full screen. You implement this method when your view controller supports two or more orientations but the content appears best in one of those orientations.
&gt;
&gt;If your view controller implements this method, then when presented, its view is shown in the preferred orientation (although it can later be rotated to another supported rotation). If &gt;you do not implement this method, the system presents the view controller using the current orientation of the status bar.

Now, let's discuss the final method. It says something about a spine location... but what is that?
The spine location is used in page controllers that use the `UIPageViewControllerTransitionStylePageCurl` style, which makes the view look like a page from a real book is turning. You might remember such behaviour from iBooks before they went for the "flat" design in iOS7.

So, by specifying the spine location, you let the page controller know how to animate the "curl" effect. By specifying `UIPageViewControllerSpineLocationMid`, for instance, you can configure UIPageViewController to show two pages at a time and make the transition look like the two pages are connected in the middle and can be turned like a real book.

### Setting UIPageViewController pages statically

**Edit 1:** _Using setViewControllers:direction:animated:completion: in such a way raises an exception in iOS8. The reason for that is that it expects the viewControllers array to have as many pages as it can display simultaneously onscreen (which is one or two depending the spine location). If you need to display a static set of pages on iOS8, consider reading my article on [building a custom UIPageViewController][Custom UIPageViewController article]
Special thanks to Pedro Borges for acknowledging that in the comment section._

As I mentioned earlier, you don't need to conform to the UIPageViewControllerDataSource protocol in order to use UIPageViewController (though you should). If you don't you can manually specify all view controllers you want to present. The downside to this is that you have to initialize all of them in advance and you won't be taking advantage of the _lazy loading_. This will probably not make much difference if you are only using a couple of pages, but imagine if you had to load a whole book... Also, the gesture-based navigation will be disabled, meaning that your users will **not** be able to swipe to go to another view.

In order to set the view controllers, you can use the following method:

- (void)setViewControllers:(NSArray *)viewControllers
direction:(UIPageViewControllerNavigationDirection)direction
animated:(BOOL)animated
completion:(void (^)(BOOL finished))completion

* The `viewControllers` parameter is an array, containing all "pages" you want to display. If you only want to set the initial view and then have your data source handle any subsequent requests, it is fine to pass an array with only that object.

* The `direction` parameter is used in order to tell the UIPageViewController in which direction it should animate the transition. The available values are `UIPageViewControllerNavigationDirectionForward` and `UIPageViewControllerNavigationDirectionReverse`. In my experience, it is not always trivial to know which direction is supposed to be used when you want to programatically change the current page, so plan ahead!

* The `animated` parameter (oddly enough) tells the UIPageController whether or not you want to animate the transition.

* The `completion` block lets you execute some code right after the tranistion is finished.

### Removing pages from UIPageController

Though it may not be all that common, there are occasions where you need to have a view controller removed from your pages. This can, of course, be achieved via the `setViewController:` method. Unfortunately, this doesn't always work all that well. More specifically, it doesn't work whenever you animate the transition and use a data source (which ironically enough is most of the time).

The problem is that UIPageViewController seems to precache its pages whenever possible and when you try to set new ones in order to effectively remove one of them, it stays in the cache and the data source methods end up not being called. Generally, you can expect that to happen for the two adjacent to the current one pages.

In order to solve that problem, you need to set the new view controllers without animating. That forces the page controller to clear its cache. If you are unwilling to loose the animation, there is a workaround that might help you. First, start the transition with animation and use the completion block in order to set the view controllers again, but this time without animating. Something like this:
<pre class="brush:objc">UIViewController* nextController = [self activePageAfterRemovingController:controller];
void (^completionBlock)(BOOL) = ^(BOOL finished)
{
// It seems that whenever the setViewControllers:
// method is called with "animated" set to YES, UIPageViewController precaches
// the view controllers retrieved from its data source meaning that
// the dismissed controller might not be removed. In such a case we
// have to force it to clear the cache by initiating a non-animated
// transition.
dispatch_async(dispatch_get_main_queue(), ^(){
[self.pageController setViewControllers:@[nextController]
direction:UIPageViewControllerNavigationDirectionForward
animated:NO
completion:nil];
});
};

[self.pageController setViewControllers:@[nextController]
direction:UIPageViewControllerNavigationDirectionForward
animated:YES
completion:completionBlock];</pre>

**Edit 2:** If your project requires dynamically adding and removing pages, I would strongly recommend switching to a custom controller. I wrote a follow-up article on [how to build a custom UIPageViewController][Custom UIPageViewController article].

I thought about creating a sample project so that you can see the page controller in action, but I quickly discovered that I don't have to. The Xcode folks are kind enough to provide a new project template containing a UIPageViewController. Just select File -&gt; New -&gt; Project -&gt; iOS -&gt; Application -&gt; Page - Based application within Xcode to access it!

![A screenshot showing the selection of the Page - Based application template in Xcode]({{ site.url }}/images/2014/04/page_controller_template.png)

I think that should be enough to get you started with UIPageViewController. Make sure to use the comment section below and let us know how **you** use UIPageViewController and maybe share some insights of your experience with it.
You can also subscribe and get notified whenever new content is uploaded.
Thanks for reading!

[Apple docs on UIPageViewController]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIPageViewControllerClassReferenceClassRef/UIPageViewControllerClassReference.html
[Apple docs on UIPageControl]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIPageControl_Class/Reference/Reference.html
[iBooks]: https://itunes.apple.com/us/app/id364709193
[Apple docs on UIpageViewControllerDataSource]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIPageViewControllerDataSourceProtocolRef/UIPageViewControllerDataSource.html#//apple_ref/occ/intf/UIPageViewControllerDataSource
[Apple docs on UIpageViewControllerDelegate]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIPageViewControllerDelegateProtocolRef/UIPageViewControllerDelegate.html#//apple_ref/occ/intf/UIPageViewControllerDelegate
[Apple docs on UIPageControl]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIPageControl_Class/Reference/Reference.html
[Custom UIPageViewController article]: http://devmonologue.com/ios/tutorials/custom-uipageviewcontroller/