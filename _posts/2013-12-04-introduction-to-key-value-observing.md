---
id: 26
title: Introduction to Key-Value Observing in iOS
date: 2013-12-04T23:04:52+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=26
permalink: /ios/introduction-to-key-value-observing/
categories:
  - iOS
  - Key-Value Coding
  - KVC
tags:
  - Key-Value
  - KVC
  - KVO
---
Today, we are going to talk about Key-Value Observing in iOS. In my <a href="http://devmonologue.com/ios/2013/12/03/using-key-value-coding-for-validation/">last post</a>, I introduced some Key-Value coding concepts. We learned about how to change an object's properties and perform validation deep inside our data model so that we can lay a solid foundation for our application's error handling. We also listed some disadvantages and potential problems so that we all know what to avoid when using this potent weapon in an iOS developer's arsenal. So if you missed it, make sure you check it out afterwards.
<h3>What is Key-Value Observing?</h3>
Key-Value Observing is functionality that allows objects to get a notification whenever a specified property in another object gets modified. This means that every time that property is changed, another object, called an "observer" gets a notification (callback). All this is packed inside NSObject so, in general, it can be used for any Objective-C class.
<h3>Why would one choose to use Key-Value Observing?</h3>
Although delegation and NSNotifications are an excellent way to get notified that something important happened in another object, this might not always be the best solution. it might also be the case that, just like with Objective-C categories, your are not able (or willing) to modify an existing object in order to add a delegate or post an NSNotification. Key-Value Coding is also a more compact and easier way to get the job done in many cases. Let's way you have a bunch of data objects and whenever, for some reason, one of those data objects changes, you have to synchronize the whole array with a server or local storage. You might think that, in the latter case you should be using CoreData - you can always get notified in case of changes like that. This is true, but consider this - CoreData will only notify you that an object has changed, and it will trigger for all changes in the object. If you wanted to synchronize only if certain properties changed, you would have to make some extra effort. Returning to the example above, you are not left with many options. It seams like an overkill (and it doesn't feel right) to have a delegate of a data class. Also, who would be the delegate? What if several objects want to be the delegate? NSNotifications fix the second problem, but it is still not a very good fix. Now, with Key-Value Observing, whoever is interested in update notifications (and has a reference to the data object) can add itself as an observer and specify which properties it is interested in. It's that simple!
<h3>Which objects support Key-Value Observing?</h3>
Almost any NSObject subclass. In fact, It is built into NSObject itself. Every time you create a property using @property, that property is eligible for key value observing. Additionally, all NSManagedObjects DO support it.
<h3>How does one register as an observer?</h3>
Just add the following code:

<code>[object addObserver:self // self will receive the callback
forKeyPath:propertyName // Changes in the property with name forKeyPath trigger callbacks
options:NSKeyValueObservingOptionNew
context:nil];</code>

Similarly, stop receiving notifications by calling:

<code>[object removeObserver:self <span style="font-family: Georgia, 'Times New Roman', 'Bitstream Charter', Times, serif">forKeyPath:propertyName];</span></code>
<h3>Where does one receive callbacks from an observer?</h3>
You need to implement the following method:

<code>- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context</code>

In it, you have access to the property name that has been changed via the "keyPath" parameter and to a reference to the object that has been changed. In this method's body, you can insert code that handles the change appropriately.

That's all you need to know to start observing!

Let me know in the comment section how do YOU use Key-Value Observers and what problems you were able to overcome with it.

Thanks for reading.