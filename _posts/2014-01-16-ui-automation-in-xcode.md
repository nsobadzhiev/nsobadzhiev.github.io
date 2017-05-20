---
id: 66
title: UIAutomation in Xcode
date: 2014-01-16T22:11:39+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=66
permalink: /debug/ui-automation-in-xcode/
categories:
  - debug
  - Testing
tags:
  - automation
  - Instruments
  - testing
  - UIAutomation
  - Xcode
---
<h3>Testing UI automatically</h3>
As you know, UI testing is tricky to do automatically, especially on mobile. On the desktop and in web development, there's Selenium for instance,  but smartphones, being embedded devices, are hard to automate. Even test driven development leaves some to be desired when it comes to the user interface. Most people have to rely on manual testing. However, there is hope! There are frameworks, such as <a href="https://www.cloudmonkeymobile.com/fonemonkey-ios">FoneMonkey</a> that allow programmers to write scripts for functionally testing their applications. But more importantly, though many people don't know it, such a tool is already integrated into Xcode itself - you can find it in the Automation section of the Xcode profiling tool.
<h3>What is UIAutomation?</h3>
UIAutomation is part of the "Instruments" application that comes bundled with Xcode. It allows developers to use JavaScript in order to write scripts that simulate actions on the user interface. This allows a large portion of the UI testing to become automatic. Using UIAutomation's language, users can create touch events (both taps and complex multi-touch gestures), locate visual components such as views and buttons, examine the view hierarchy, create screenshots, check error conditions using predicates and log test results. While it does not provide the functionality of a unit test, it can definitely help you achieve a considerably higher degree of automation. As I mentioned earlier, there are other tools for automating user interface testing, but they cannot achieve the amount of integration and ease of use provided by UIAutomation. FoneMonkey, for instance, requires you to create a separate build target containing all of it's libraries, whereas with UIAutomation you only hit the "Profile" button (instead of the "Run" button) and you're all set. Now, in FoneMonkey's defense, it is clear that Apple will never allow third parties to have the amount of access needed to create a fully integrated testing suite, so needless to say, the stock option, provided by Xcode will always be the winner in this category.
<h3>How does UIAutomation work?</h3>
<em>This section is dedicated to some core UIAutomation principles. Though you can skip it if you only want to see some examples, I would recommend reading it so that you know how all parts fit together.</em>

The first thing we should cover is the way UIAutomation identifies component in the user interface. A major strength of this tool is that your script doesn't normally consist of  taps at specific coordinates. In fact, most of the time (when you're not working with complex touch gestures) you don't have to bother with coordinates at all. What you do is, you specify a view component, and you simulate a touch event to it. This makes your scripts very flexible in the sense that, if you change the layout of your screens or maybe modify it to fit another resolution like the iPad's, you don't have to rewrite your scripts. As long as a given component is still present on the screen, the script will find it and continue going. So how does UIAutomation achieve that?

All components in an UIAutomation script are identified by their accessibility label. These can be set both in Interface Builder and programatically. Normally, this label is used when providing accessibility features to an application, but here it doubles as the "name" of your view in the automation script's scope.

<strong>Note:</strong> <em>You don't always need to have the accessibility label set. There are other ways to identify a view. The label just makes it more consistent.</em>
<h4>Setting accessibility labels in Interface Builder</h4>
In Interface Builder, each view's accessibility label can be easily set in the "identity inspector" - it is the third tab in the right assistant editor:

[caption id="attachment_69" align="aligncenter" width="269"]<a href="{{ site.url }}/images/2014/01/identityInspector.png"><img class=" wp-image-69 " alt="Accessibility options in Interface Builder" src="{{ site.url }}/images/2014/01/identityInspector-384x1024.png" width="269" height="717" /></a> Accessibility options in Interface Builder[/caption]

It is important to make sure the "Enabled" option is set. Afterwards, you can choose an appropriate name for your view. You should also think of something that you can be actually useful for accessibility, while you're at it.
<h4>Setting accessibility labels programmatically</h4>
If you're not using Interface builder, you can add accessibility labels to your views with the following code:
<pre class="brush:objc">testedButton.accessibilityEnabled = YES;
testedButton.accessibilityLabel = @"Big red button";</pre>
That's it.
<h3>How to start UIAutomation?</h3>
As I said, UIAutomation is part of Xcode's Instruments tool. In order to start it, you need to run your application using the "Profile option" (accessed in Product -&gt; Profile, in the toolbar or by pressing CMD + I). This will rebuild your application and launch the Instruments app. You should be presented with something like this:

[caption id="attachment_70" align="aligncenter" width="605"]<a href="{{ site.url }}/images/2014/01/Instruments.png"><img class="size-large wp-image-70" alt="Instruments" src="{{ site.url }}/images/2014/01/Instruments-1024x846.png" width="605" height="499" /></a> Figure 1: The UIAutomation "New" window[/caption]

As you can see, the Instruments app provides a wide range of tools for tweaking your application. If you haven't already, I suggest that you take the time to get to know (and love) them. But for now, let's choose the "Automation" option and start writing scripts.

[caption id="attachment_71" align="aligncenter" width="605"]<a href="{{ site.url }}/images/2014/01/UIAutomationHome.png"><img class="size-large wp-image-71" alt="UIAutomation's home screen" src="{{ site.url }}/images/2014/01/UIAutomationHome-1024x620.png" width="605" height="366" /></a> Figure 2: UIAutomation's home screen[/caption]

This is where the magic happens. By the time you see this screen, the application should have started running. UIAutomation can be used on both the simulator and on a real device, though the simulator has a few limitations (it cannot take screenshots) but more on that later. In the screenshot above we can see that Apple's GenericKeychain example has been running for 7 seconds now, and there is no script currently automating. It even tells as that an error has occurred while trying to run it. This is just Xcode's fancy way of saying that there is no script to start currently.

As you can see, there is quite a lot going on in UIAutomation's window. There are timers, timelines, recording buttons and a lot more. However, at least for now, all you're going to need is the trace log, where you'll be looking at your script running (and failing miserably), and the source code editor, where you'll be writing your JavaScript. If that scares you as much as it scared me, don't worry - the Javascript we will be using is fairly straight-forward and self-explanatory.
<h3>How to create a script in UIAutomation?</h3>
In my opinion, this is where UIAutomation starts to fall short. It's user interface is just not intuitive enough. It supports running only a single script, several open files are not indicated well enough in the view and switching between the source editor, trace log and editor log is a mess. It's not a big deal, but being bundled with the Xcode IDE, I expected a lot more.

Anyway, let's create a new script. In the middle of the left pane, you will see an "Add" button. Clicking it will show a drop down menu allowing you to create, import or open a recent script. Figure 4 depicts just that:

[caption id="attachment_75" align="aligncenter" width="605"]<a href="{{ site.url }}/images/2014/01/UIAutomationEditorScript.png"><img class="size-large wp-image-75" alt="The UIAutomation window. &quot;1&quot; creates a new script and &quot;2&quot; runs the selected one" src="{{ site.url }}/images/2014/01/UIAutomationEditorScript-1024x620.png" width="605" height="366" /></a> Figure 4: The UIAutomation window. "1" creates a new script and "2" runs the selected one[/caption]

[caption id="attachment_76" align="aligncenter" width="548"]<a href="{{ site.url }}/images/2014/01/UIAutomationEditorSwitch.png"><img class=" wp-image-76  " alt="Show the slightly unintuitive way to navigate between script, editor and trace log perspective" src="{{ site.url }}/images/2014/01/UIAutomationEditorSwitch.png" width="548" height="354" /></a> Figure 5: Show the slightly unintuitive way to navigate between script, editor and trace log perspective[/caption]

After you do that, you should be presented with a source code editor and the following statement already inserted for you
<pre class="brush:js">var target = UIATarget.localTarget();</pre>
Figure 4 also shows the way scripts are run. Press the "Play" button at the bottom of the source editor and UIAutomation will immediately start executing. It is important to note that your application will NOT be rebuilt or re-run - the script will start from wherever state you left your app at. So make sure you navigate the user interface to whatever screen you expect in your JavaScript.
<h3>How to write UIAutomation scripts?</h3>
The reality is that UIAutomation is not as well documented as other iOS frameworks and tools. <a href="https://developer.apple.com/library/ios/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/UsingtheAutomationInstrument/UsingtheAutomationInstrument.html">Apple's documentation</a> only shows a few generic examples and does not go into great detail explaining UIAutomation's features. However, using this article and the <a href="https://developer.apple.com/library/ios/documentation/DeveloperTools/Reference/UIAutomationRef/_index.html#//apple_ref/doc/uid/TP40009771">Javascript API's reference</a>, you should be able to learn quickly. Here, we will go over all important principles of writing UIAutomation scripts and after that you should be able to start creating your own by looking up the classes in the API reference.

Let's start from the beginning. What is that strange piece of code UIAutomation put at the start of my newly created script? "UIATarget.localTarget();", most commonly seen as "UIATarget.localTarget().frontMostApp().mainWindow()" is the way you get a reference to the screen, currently displayed on the device. It should be the the view controller you plan on testing.
<h4>Accessing elements in the view hierarchy</h4>
The most important task for your script will be accessing different UI components. As mentioned earlier, UIAutomation is not about tapping on certain coordinates, but identifying views and changing their properties. If you are in doubt where a certain component is located within the view structure, you can use the following code to print it in a convenient manner:
<pre class="brush:js">element.logElementTree();</pre>
This will print all components in a tree-like hierarchy inside the trace log and even create a screenshot of each view. This makes debugging your script very visual and user-friendly. Unfortunately, creating screenshots is not available when using the simulator.

You can access an element's "children" using several methods provided by the element's class. Here's some examples of such methods:
<ul>
	<li>tableViews()</li>
	<li>cells()</li>
	<li>textFields()</li>
	<li>secureTextFields()</li>
	<li>buttons()</li>
	<li>tabBar()</li>
	<li>etc.</li>
</ul>
Since you have a reference to the currently displayed view controller, you can start going down the view hierarchy. You can also go upwards, accessing an element's superview:
<pre class="brush:js">element.parent();</pre>
As you can see, most of these methods return an array, containing all components that match the requested criteria (for instance, all buttons in the view). If you are unfamiliar with JavaScript, here's how to access an element of this array:
<pre class="brush:js">element.buttons()[2];</pre>
Now, you might say that just hardcoding that "2" in not a very flexible thing to do. What if a new button is added... or the others rearranged? Well, remember that thing about the accessibility labels above? You can also refer to components by them:
<pre class="brush:js">element.buttons()["Login Button"];</pre>
Here, we specifically said that we wanted the button, named "Login button". In my experience, the name does not necessarily have to be the accessibility label. It can be the button's title label value. In fact, if you don't want to modify your existing source code, you might just use that. However, button titles are subject to localization and possibly branding and you might be better off using the accessibility (if it's not localized as well).

<em>I would like to emphasis something that caused me frustration a while ago. There is a difference between a text field and a <strong>secure </strong>text field. You cannot access a secure text field by calling "element.textFields()", you have to use "element.secureTextFields()".</em>
<h4>Generating touch events</h4>
Naturally, the most important job of a user interface automation framework is to generate input events. Fortunately,  this is made easy with UIAutomation. I guess generating tap gestures is good enough for most people. Here's how it's done:
<pre class="brush:js">UIATarget.localTarget().frontMostApp().mainWindow().tabBar().buttons()[4].tap();</pre>
As you can see, it is just a matter of calling the "tap" method of an element. The code example above sends a single finger tap event to the fifth button in a tab bar.

UIAutomation can do a lot more than simple taps. It can perform complex multitouch gestures. I'm not going to go into details about them, but you can read about them from <a href="https://developer.apple.com/library/ios/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/UsingtheAutomationInstrument/UsingtheAutomationInstrument.html#//apple_ref/doc/uid/TP40004652-CH20-SW96">Apple's documentation</a>.
<h4>Introducing delays between UIAutomation actions</h4>
First of all, maybe you should know a little more about the way UIAutomation schedules actions. It is actually very simple - tries to execute an action and if your application is not in a state that can accept this sort of event, it will wait a specified amount of time. If it is unable to complete the action and this timeout expires, the test fails.

This timeout can be changed during runtime using the following code:
<pre class="brush:js">UIATarget.localTarget().pushTimeout(15);</pre>
And to revert that change, all you do is:
<pre class="brush:js">UIATarget.localTarget().popTimeout();</pre>
In the example, we changed the timeout between actions to 15 seconds and then reverted to the default value, which should be 5 seconds. Five seconds, in my experience seems to be good enough for most cases, but I guess that if you will be waiting for data coming from the network, you might want to consider increasing that value.
<h4>Logging information in UIAutomation's trace log</h4>
Naturally, UIAutomation provides support for logging arbitrary information in the trace log. Most actions that your scripts do are already logged, but it is always nice to be able to provide additional information for debugging. Especially since UIAutomation does not allow you to run multiple scripts, you will need a way to log which test you are running in order to know what went wrong if the script fails. Available <a href="https://developer.apple.com/library/ios/documentation/ToolsLanguages/Reference/UIALoggerClassReference/UIALogger/UIALogger.html#//apple_ref/javascript/instm/UIALogger/logMessage">here</a> is a reference of all logging methods supported. Overall you would be using "logFail" and "logPass" to indicate the test result, "logStart" to mark the beginning of a new test and "logMessage" (or any other method with different log level) to write important data into the trace. Unfortunately, unlike it's counterpart from Objective-C, NSLog, UIALogger does not provide a printf-like signature, but you can use the following to, at least, concatenate objects into the log message:
<pre class="brush:js">UIALogger.logPass("Hello " + "world");</pre>
<em>I just had to include a "Hello world" reference. </em>Also, the Javascript language includes a "sprintf" function, but I guess it is not available here.

Another useful feature of UIALogger is the ability to create screenshot. This only works on iOS devices and not the simulator. Try this:
<pre class="brush:js">UIATarget.localTarget().captureScreenWithName("ScreenShot");</pre>
An image of the currently displayed screen should appear right into the trace log. Very useful if a test fails and you want to save whatever state the user interface was in while it happened.

Figure 6 show what a typical trace log looks like.

&nbsp;

[caption id="attachment_79" align="aligncenter" width="605"]<a href="{{ site.url }}/images/2014/01/UIAutomationTraceLog.png"><img class="size-large wp-image-79" alt="A trace log showing several failed tests " src="{{ site.url }}/images/2014/01/UIAutomationTraceLog-1024x500.png" width="605" height="295" /></a> Figure 6. A trace log showing several failed tests[/caption]
<h4>Verifying test results</h4>
Since UIAutomation is a testing platform, obviously it is going to need a way to verify that the application being tested is acting the way it should. Unfortunately, there isn't an integrated way to verify test results. For the most part you will have to come up with your own pass criteria and perform checks manually.

<strong>Example:</strong> <em>Did the application login? Well, how do you define a successful login? The isLoggedIn method returns YES? It does, but you're not debugging and you don't have access to the isLoggedIn method in your script. Hmmm...</em>

In the example above your verification will have to check that your user interface shows the screen that is displayed right after login. Something like "The client zone screen appears" or "the tab bar is now visible". Choosing a criteria is not always easy. You have to choose something that will ALWAYS be true for every successful login, but false for every failed attempt. Also it has to be something that will still work after same UI changes are made. For the most part test result verification looks something like this:
<pre class="brush:js">var logoutButton = UIATarget.localTarget().frontMostApp().mainWindow().buttons()["Logout"];

if (logoutButton()) {
    UIALogger.logPass("Application was able to login");
}
else {
    UIALogger.logFail("Login failed");
}</pre>
Here, the script tries to get a reference to a button named "Logout" inside the screen's view hierarchy. The test is considered to be have passed if such a button exists.

To make your life easier, UIAutomation has support for using predicates. It also uses the same syntax used for NSPredicate:
<pre class="brush:js">var textField = screen.textFields()[0].firstWithPredicate("value is ‘Username’");</pre>
Here, predicates are used in order to find the right text field. Of course, the example is overly simplistic but using predicates, you will be able to verify complex conditions.
<h4>Handling alerts</h4>
While testing on a real devices, execution might be interrupted by unexpected events such as receiving a call, a local notification or even a low battery warning. Also, your application will probably be using some alerts of it's own. UIAutomation provides functionality for handling these alerts. Intercepting an alert can be done in the following way:
<pre class="brush:js">UIATarget.onAlert = function onAlert(alert){
    var title = alert.name();
    UIALogger.logWarning("Alert with title ’" + title + "’ encountered!");
    return false;
}</pre>
Every time an alert is shown during the execution of your script, this function will be called. If it returns false, the alert will be dismissed automatically by tapping the cancel button, and you can perform some additional actions provided that you require some additional processing:
<pre class="brush:js">UIATarget.onAlert = function onAlert(alert) {
    var title = alert.name();
    if (title == "This error") {
        alert.buttons()["Ignore"].tap();
        return true;
    }
    return false;
}</pre>
This demonstrates that you can decide not to ignore the alert altogether, but decide which option to choose and tap that button.
<h3>Conclusion</h3>
UIAutomation is available with Xcode 4.5 and later and is a tool that definitely deserves your attention. Use the comment section to let me know - will you be using UIAutomation in the future and do you plan on teaching your QA team write automation scripts?

Thanks for reading!