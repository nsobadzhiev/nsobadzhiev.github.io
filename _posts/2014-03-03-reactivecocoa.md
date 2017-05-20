---
id: 150
title: ReactiveCocoa
date: 2014-03-03T17:40:06+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=150
permalink: /frameworks/reactivecocoa/
categories:
  - frameworks
tags:
  - cocoa frameworks
  - functional programming
  - ios
  - ObjC
  - Objective-C
  - reactivecocoa
---
[caption id="attachment_152" align="aligncenter" width="243"]<a href="{{ site.url }}/images/2014/03/cocoa_cup.jpg"><img class="size-full wp-image-152" alt="The image shows a cup of cocoa and a piece of paper with a drawing on it that serves as a logo for the Cocoa framework" src="{{ site.url }}/images/2014/03/cocoa_cup.jpg" width="243" height="179" /></a> The Cocoa framework[/caption]
<h2>ReactiveCocoa first steps</h2>
So what is <a href="https://github.com/ReactiveCocoa/ReactiveCocoa/tree/master/ReactiveCocoaFramework/ReactiveCocoa">ReactiveCocoa</a> and why would one have first steps with it? ReactiveCocoa is an Objective-C framework that introduces a selection of features from functional programming (more on that later). More specifically, it treats programs as contiguous conversion of input to output data. This means that it takes a stream of data, performs some transformations on it and returns the modified information as a result. This might seem like a bit of an understatement (obviously programs do a lot more than convert data, especially nowadays) but when you think about it, you will realize that it is completely true. The definitions of input and output data might have become more vague and abstract, but essentially this is what applications do. That being said, what is the benefit of perceiving programming that way?

With the addition of many, often asynchronous, inputs and outputs, it is increasingly difficult to manage all information bloating around in our applications. Handling multiple concurrent operations and responding to different events throughout the program introduces a lot of complexity in the code. Most of it are "if" statements and checking states. This makes it harder to read and understand. And this is where ReactiveCocoa comes in.

Since ReactiveCocoa regards incoming events as streams of data, it is able to handle incoming data in a simpler, more straight-forward way, without the need of constantly checking states. The logic flows more natural and streamlined.
<h2>Declarative programming</h2>
As I mentioned earlier, ReactiveCocoa brings a lot of concepts from functional (declarative) programming to Objective-C. It is time to talk about what functional programming is and how it differs from the "conventional" languages.

These conventional language like C, Java and Objective-C are called procedural languages. This means that by using them, you provide specific steps the computer has to make in order to accomplish the task. When implementing an algorithm, you provide detailed description of the way the computer should find the solution. In contrast, declarative languages describe what result they want to achieve without instructing the computer how to find it. In essence, declarative programming provides an answer to the question "What?" and procedural programming answers to the question "How?".

Declarative programming describes functionality in a matter that is closer to the human logic. This allows developers to write code that is more natural to the way they think. Those that have a little experience with it (like me) will agree that there is nothing natural or easy about functional languages. However, others that have a lot of experience in the field will probably argue that they allow them to express ideas in a more elegant and streamlined way. In my opinion, declarative programming is something that requires a little bit more time to get used to (after all, procedural programming also looks intimidating when you first start studying it). And that's exactly how I feel about ReactiveCocoa as well.
<h2>Some functional programming concepts</h2>
As mentioned, functional programming is not about writing the exact steps the program has to go through in order to get the task done, but rather it describes what result is it looking for and relies on the language itself to satisfy the conditions. This is an interesting concept to say the least, but we don't need to get in too deep with it as it's not really that important for understanding ReactiveCocoa. Instead, let's talk about some features that it makes use of:

<span style="text-decoration: underline;"><b>Streams:</b></span>

Streams are effectively the NSArray of functional programming, but with some interesting differences. First of all, they are more like a list, than an array because their elements cannot be accessed directly. You only have access to the next element. Also, they often make use of <b>lazy loading</b>. This means that values are not evaluated when the stream is created but rather when they are needed – at the time of their first accessing. Additionally, loading is performed only once – if you access an element twice, it's only evaluated the first time. Secondly, streams have the ability to be infinite. Since they are not loaded when they are first created, there is nothing stopping you from creating one with no end – elements are just added when they are needed.

<span style="text-decoration: underline;"><b>Map, filter:</b></span>

Functions like map and filter (and others) are nothing special, but they are typically found in functional programming paradigms. Such can also by found in languages like Python, Perl, Ruby and now... Objective-C through ReactiveCocoa. They are used on collections (streams, NSArray, etc.) to transform the values using some condition. The Map function, for example, takes each element of the collection and changes it according to a rule. The rule is a piece of code (a block in the ObjC case) that specifies how the modified value is calculated. The filter function on the other hand removes some elements of the collection using a condition specified in it's parameters. For instance, it can be used on an array containing [1, 2, 3, 4, 5, 6, 7, 8, 9] with the (value % 2 == 1) condition in order to get the filtered array [1, 3, 5, 7, 9].
<h2>The ReactiveCocoa framework</h2>
When you first open ReactiveCocoa's github page, unless you have experience with functional programming or at least Python, Ruby, Perl or something similar, you will probably have a hard time understanding what any if it does. It features some really strange synthax with blocks and terms like "sequence", "stream" and "subscriptions". Everything seems to have an alternative meaning. Indeed, initially it is really hard to read ReactiveCocoa code. It almost looks as if it's not Objective-C. Many of you will, no doubt, reconsider using the framework because of that. Let's face it – even if you take the time to learn how to use it, can you risk writing code that others will need so much reading to understand. Well, if you really think that, it's a real shame because ReactiveCocoa can potentially be a hidden treasure.

<i>ReactiveCocoa makes heavy use of block</i><i>s</i><i> and Key Value Observing (or KVO). If you don't know much about it, I wrote a couple of post</i><i>s</i><i> about <a href="http://devmonologue.com/ios/ios/introduction-to-key-value-observing/">KVO</a> and <a href="http://devmonologue.com/ios/ios/using-key-value-coding-for-validation/">KVO validation</a>.</i>

So what are the main concepts behind the framework? The main issue that it addresses is providing a centralized method of managing multiple asynchronous tasks. These include network and file operations, lengthy processes, user input events and UI updates. ReactiveCocoa provides streamlined means of creating, observing, serializing and synchronizing them. Like blocks, it also allows developers to describe non-blocking behavior in a single place in the code. Normally, if you want to execute a network operation without blocking the thread, you have to rely on delegation. This means that the request is started in one place and the callback received somewhere else in the source code. This makes it a little harder to read. With blocks, and ReactiveCocoa, the problem is resolved by having the callback right after the network operation creation.

Similar to blocks in Objective-C, ReactiveCocoa allows you to initiate operations and specify how they will be handled once they are complete using a common interface. But it also introduces the ability to chain operations together, execute them concurrently and easily stop them whenever an error occurs. This way, an application becomes a series of action-reaction pairs, or as I wrote in the beginning of the article a "<i>contiguous conversion of input to output"</i>.
<h2>ReactiveCocoa documentation</h2>
Reactive cocoa has a <a href="https://github.com/ReactiveCocoa/ReactiveCocoa/tree/master/Documentation">documentation</a> in it's github repository. It's absolutely critical that you read it, especially the <a href="https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/Documentation/FrameworkOverview.md">framework overview</a> and the <a href="https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/Documentation/BasicOperators.md">basic operators</a> example, otherwise you wont be able to understand how to use it. Also, I think that it's really helpful (if not mandatory) to read the source code. It has a lot of doxygen style comments and is very well organized so it is an excellent place to go to for information.

<span style="text-decoration: underline;"><b>Streams</b></span>

ReactiveCocoa's equivalent to the streams from functional programming that we talked about are called sequences. All transformation functions like map, filter, combine and reduce are available when working with sequences. Some of those functions we discussed earlier and some we didn't, but they are pretty self explanatory.

Since they are so close, ReactiveCocoa makes it convenient to create sequences from NSArray. This is done using a category (NSArray+RACSequenceAdditions) on the array class. The whole framework makes heavy use of categories in order to provide easy access to ReactiveCocoa functionality for many Foundation classes like UIControl, NSArray, NSDictionary and NSString. Here's an example of creating a sequence and transforming it using the map function:
<pre class="brush:objc">RACSequence *sentence = [NSArray arrayWithObjects:@”I”, @”love”, @”chicken”].rac_sequence;

RACSequence *sentenceAfterMap = [sentence map:^(NSString *word) {

return [word stringByAppendingString:@” ”];

}];</pre>
The first line demonstrates how easy creating a sequence is. Given that you have an array containing all the values that you need, all you do is call the "rac_sequence" property getter in order to get a newly created RACSequence object. The second line uses the map function to modify each entry in the sequence by appending a whitespace after it.

This example was not that impressive and you are unlikely to use ReactiveCocoa for something like that but nevertheless it's worth noting what many of the features in the framework are based on.

<span style="text-decoration: underline;"><b>Responding to UI events</b></span>

A more meaningful use of ReactiveCocoa should be responding to user interface events. Here it is:
<pre class="brush:objc">[[myButton rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(UIButton *sender) {

// perform some action

}];</pre>
Again, thanks to the categories ReactiveCocoa created on UIControl, we are able to create a signal using a single line of code. We specify that we want to create a signal that activates every time the button registers a "UIControlEventTouchUpInside" event and executes the provided block.

However, often when the user taps a button on the UI and start a potentially long running task, you might want to disable that button to avoid the user tapping it again. This means that in addition to requiring control over the buttons action, it would also be helpful to be able to control some aspects of the control itself. Luckily enough, this is easily achievable with ReactiveCocoa:
<pre class="brush:objc">RAC(myButton, enabled) = [RACSignal combineLatest:@[

self.myUISwitch.rac_newOnChannel,

RACObserve(someObject, shouldEnable),

] reduce:^(NSNumber* switchValue, NSNumber *shouldEnable) {

return @(switchValue == YES &amp;&amp; shouldEnable .boolValue == YES);

}];</pre>
Here, we instruct ReactiveCocoa to change myButton's "enabled" property by getting the latest values from myUISwitch and someObject.shouldEnable and calling the reduce block to determine the right value. There are a few things to consider in this code. First of all, the reduce block is used to filter values that we don't need. For instance, it will return NO every time it is called and the switch is not on. Of all the possible values it only allows combinations that require myButton to be active.

Secondly, you might wonder how we created a combination of both signals – one for the switch and one for the "shouldEnable" property in "someObject". Well, the switch was easy because ReactiveCocoa provides a category on UISwitch – UISwitch+RACSignalSupport that provides an easy way to get a signal that will fire every time the control's value is changed. However, "someObject" is an instance of a class that probably does not have such a property, so we needed to use Key-Value observing to get notified whenever the "shouldEnable" property is changed. This is done via the "RACObserve" method in ReactiveCocoa.

Finally, a new method that we never explain in the example is RAC. It's just a convenience method for creating the binding, or as the documentation describes it - "RAC() is a macro that makes the binding look nicer."

So, what is a signal anyway? According to the documentation:
<blockquote>A signal, represented by the RACSignal class, is a push-driven stream</blockquote>
Yeah... It's a bit obscure, but intuitively it is a collection of event. The trick is that as new event are generated (in the example above, a new tap is registered), all subscribers are notified by calling the block that was provided. That's what <b>push </b>means in the definition above – the signal will make sure all subscribers are notified. Subscribers?

A subscriber in ReactiveCocoa is what a delegate object that conforms to a given protocol is in Objective-C. RACSubscriber itself is a protocol / base class that describes the methods that are called whenever a next event is registered. These are:
<pre class="brush:objc">- (void)sendNext:(id)value;
- (void)sendError:(NSError *)error;
- (void)sendCompleted;
- (void)didSubscribeWithDisposable:(RACDisposable *)disposable;</pre>
You might have a hard time understanding what all these methods are for, but that's because you should know a thing or two about how signals work. As mentioned, they are streams of events. These events are chained together. When one of them completes, the subscribers are notified and the signal continues with the next one. Once all tasks are successfully executed, the "sendCompleted" message is sent. If an error occurs during one of the events, the "sendError" method is invoked and all subsequent tasks are cancelled. Conveniently enough, once the signal has finished, it is automatically released so you generally don't need to be concerned with memory management.

Another important ReactiveCocoa class is RACDisposable. It is the base class that encapsulates the logic for canceling a subscription. Above I mentioned that a signal stops all remaining events once it encounters an error. But how does it know how to stop a particular task when they can be so different. A file system operation is cancelled in a different way than a network operation or a complex calculation? The answer is that RACDisposable creates a common interface that subclasses have to conform to in order for the signal to be able to cancel them successfully if needed. I don't think the average developer will need to use this class directly that often, but nevertheless, it is an important part of ReactiveCocoa's internals.

So, let's see what ReactiveCocoa does when faced with the task of performing several tasks concurrently. Consider this scenario. You have to start several network requests and when (and if) all of them are successful, perform some action. This is relatively hard and messy to do in Objective-C. You have to deal with synchronization, your code will get messy due to all this asynchronous execution and generally become hard to read. ReactiveCocoa deals with that by managing synchronization for you and allowing you to write your code in such a way that it's easy to immediately understand what you are trying to achieve. Before I show you how it's done, let's see how a single request is started:
<pre class="brush:objc">self.myRequest = [[RACCommand alloc] initWithSignalBlock:^(id sender){
    return [self startRequest];
}];

[self.myRequest.executionSignals subscribeNext:^(RACSignal *mySignal){
        [mySignal subscribeCompleted:^ {
        NSLog(@"Request is successfully!");
    }];
}];</pre>
First of all, a RACCommand is created that contains the code that will start the network request. After that, the callback for when the request is completed is set using the “executionSignals” property. In this example, only the “subscribeCompleted” callback is set and the error events are ignored for simplicity. Once the signal is created, it can be linked to a button so that the request is started whenever the button is tapped:
<pre class="brush:objc"> startButton.rac_command = self.myRequest;</pre>
Or, it can be started manually by invoking RACCommand's execute method.

Now, to start several requests at the same time, all you do is:
<pre class="brush:objc">[[RACSignal merge:@[[self myRequest1], [self myRequest2]]] subscribeCompleted:^{
    // Do something.
}];</pre>
Here's why I explained streams earlier. We had to use them in order to combine (or merge) the two commands and then, set a callback for when they both complete.

The final thing I'm going to show you is chaining requests so that they are started one by one:
<pre class="brush:objc">[[[self startRequest1] flattenMap:^(NSDictionary* resultDict){
    return [self startRequest2];
  }]
                       flattenMap:^(NSDictionary* resultDict){
    return [self startRequest3];
  }]
                    subscribeNext:^(NSArray* serverDataArray){
    // do something after the second request
  } completed:^{
    // do something after the third request
}];</pre>
The important thing here is the flattenMap method. It can be used to transform each value into a new stream. After that, they will be combined into a single object, thus making a stream containing both of our commands.

Conclusion

ReactiveCocoa in an interesting hybrid between declarative and procedural programming. It attempts to bring the benefit of functional languages to Objective-C. Like all of them it features some hard to understand synthax that most developments have problems with. However, every beginning is tough. Even procedural languages are difficult at first. Remember when you first saw Objective-C code? Didn't it look weird? But once you got used to it, you discovered that it is actually quite intuitive and logical.

The point is that we are accustomed to C-style languages and refuse to use anything that does not share the same concepts. But we shouldn't stop learning. ReactiveCocoa has the credentials of becoming the next big thing and we shouldn't dismiss it just because it looks strange.

Once again, thanks for reading!