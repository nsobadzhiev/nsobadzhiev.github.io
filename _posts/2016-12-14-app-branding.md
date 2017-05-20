---
id: 468
title: App Branding
date: 2016-12-14T16:03:25+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=468
permalink: /ios/app-branding/
categories:
  - iOS
tags:
  - app branding
  - ios
  - UIAppearance
---
# App branding and UIAppearance

## Introduction

Supporting several application variations sharing the same code base is a programming topic that affects lots of developers. Especially when you think back several years ago with the explosion of holiday themed games, lite and premium version apps and re-skinning. Not that companies have stopped doing those things, but it seems, at least to me, that it's no longer that prominent.

But it's not only that. We also have white label solutions and enterprise apps specifically tailored for a company. The point is that there are lots of uses cases for apps that need to change at build time according to external requirements, and share 99% of their code at the same time.

The problem with programming though, is that 99% still means you have to support two distinct configurations. Also, what starts off as 99% similar tends to slowly diverge to 95, then 90 and then even less.

There are a lot of ways to tackle this app branding problem. And I don't think anyone is in a position to say which way is the correct or the best one. It all depends on the specific circumstances of the project at hand. It's all about anticipating the scope of the version differences and in what direction requirements might shift in the future.

However, even with that in mind, there are certain best practices and platform features we can leverage to have a more successful branding implementation.

## Techniques

We are going to go over several techniques that can be used to achieve app branding. They are all not perfect, but can be useful tools in the right circumstances. These are:

* Branching
* Build Targets
* Subclassing
* Configurations
* UIAppearance

Often times several of those items will have to work together, in order to get the job done. Especially build targets are paramount to most solutions. By themselves, they are rarely enough, but coupled with others, they are probably the most powerful tool for app branding.

### Branching

In a source controlled environment, you might be tempted to create a different branch for every branded version of your application. Especially if the new version requires a lot of changes, it might be easier to move to a new branch and implement changes without worrying too much about the main project - you cannot break your generic app if you're working in a different branch. However, the price for this decision is quite big. With time your branches will diverge a lot. And even if they didn't, keeping them in sync as new features and fixes get added, would be tedious and error-prone.

Another similar idea would be to extract the common code into a framework, submodule or whatever is convenient for the particular case, and then have different projects, using those components and implementing the different, "brand-specific" parts. This might seem like a nice idea, and if done correctly might be genuinely awesome, but it is, in my opinion, difficult to achieve. For one thing, the distinction between "common" and "brand-specific" is too vague. There is rarely a single inflex point where you can draw the line. So often, you will have either too little "common" code and end up duplicating features in the "brand-specific" section, or the opposite - you will have too much specifics in you "common" code, leaving you desperate to find ways to change that behavior depending on a version.

So unless you have a really good idea of how you might separate the code and how new requirement might affect that idea, I think you should treat this approach with a lot of suspicions.

### Build Targets

This is Xcode specific, but every IDE should have a way to achieve something similar. As you probably know, an Xcode project can have several targets. The ones you usually see, are targets for the application, for unit tests and for UI Tests. But of course, you can add your own. Each target has different build settings and can work with a different subset of source files. What makes targets perfect for app branding is the fact that you can use different `info.plists` in order to have different images, bundle IDs etc, different source files (you can choose which source file is included in which targets), different artwork, different Interface builder files etc.

This makes it really easy to have different resources for your different app variations. You can even have files with the same name, but included in different targets. In fact, this is a common solution to branding problems. On the other hand, managing which item goes to which target is a bit error-prone. Especially after a while, when app branding is not something you actively think about during feature implementation, it is easy to make a mistake which is going to lead to obscure build error or even worse, crashes and bugs.

So, even though targets are a powerful tool, they have their caveats so you should take special care while using them.

There's still a lot to discuss about build targets, however most will come up while discussing the next app branding techniques. So lets continue...

### Subclassing

The idea behind subclassing is simple. You create a generic (maybe even abstract) class that implements an interface for a particular feature. Then, you create a subclass for every app variation you need. This subclass actually implements the functionality necessary for that variation. Of course, then you need to make sure the right subclass is used. It might be some sort of factory method, or what we just discussed - targets.

At first, subclassing seems like a perfect option. In fact, that's why object-oriented languages have subclassing - to extend a base implementation. But upon closer examination, you often see that this is not true. To the contrary - you are going against OOP principals. Especially [Liskov's substitution principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle). Extending the class quickly turns into slaughtering the base implementation and constantly trying to do something completely different in the subclasses. At some point every class in the hierarchy is so different that the only reason for them to co-exist is because they should get called at the same locations in code. And that's where it gets ugly.

At least that's my experience. Maybe I'm doing something wrong, but up until now I've failed to see a well working app branding solution that uses subclassing extensively.

That being said, subclassing is sometimes unavoidable. And it can generally be ok as long as it's kept to a minimum.

Another variation to the subclassing technique (that I like better) is using classes with the same name, but in different targets. This is now especially easy in Swift since you most often don't need import statements. What I like about this variation is that it completely ditches the idea of inheritance. It acknowledges the fact that those classes are not really similar so you don't have to constantly fight against inheriting some base functionality. It's just 2 (or more) classes with completely different implementation, that happen to have the same name.

### Configurations

Configurations are not like the previous techniques we discussed. In fact, they often rely on subclassing and build targets for their functioning. The basic idea is to hold constants that toggle functionality and hold values for properties, specific to an app variation. But that implies having different configurations for each variation. And to differentiate between them, of course you need something like targets and/or subclassing.

On the implementation level, a "configuration" can be, for instance, a XML, JSON, plist file (essentially a resource file) or a source file full or constants (in Swift probably a file full of structures and enums). Then, the easiest way would be to have the different files only in their respective target.

### UIAppearance

Now we are getting to the good stuff. And also the bad stuff. Definitely the interesting stuff.

For a general overview of what UIAppearance does, check out the [UIAppearance reference page](https://developer.apple.com/reference/uikit/uiappearance). But in short, UIAppearance is a protocol that allows you to change the default values lots of the UI classes in UIKit use to visualize themselves. The way that works is that you, for instance, specify that the text color for UILabels is green. And from then on, every UILabel instance that is created, gets that green color by default. Pretty neat!

**Note:** UIAppearance sets the default value, but if the same property is changed after that, the value gets overridden. And while this rule makes total sense, it is sometimes annoying and creates subtile bugs. For instance, if you set the text color of a label in Interface Builder, the UIAppearance value is overridden because Interface builder applies its changes **after** the styling from UIAppearance is done. You need to make sure to leave the default option in Interface Builder.

UIAppearance is really nice for applying application-wide values. Much like setting the `tintColor` value in your root view, but much more flexible and complicated. And that's what makes it great for branding. Especially if you what to change your app's theme color, UIAppearance is a great option.

#### The good

Think about it - if you had to change a color or something similar in your whole application, it would be a nightmare. Tracking down every view that uses it, always missing some cases. With UIAppearance, you only need to set that color once, and then it will apply for every instance of that particular class after that. Something as simple as this:

```swift
UILabel.appearance().textColor = themeColor
```

You can even go one step further and extract those values in a resources file - a plist for instance. Then, as your application is loaded, you can read that file and apply the styling. That could be really nice, although I don't think a situation where you can give that resource to your non-tech colleagues and have them change it themselves, is realistic. I just think that at least a minimal tech background is needed to grasp the concept of what works and what doesn't.

#### The bad

UIAppearance is no doubt a great idea. But it's not for the short tempered, at least not in the beginning. There's just a lot of subtleties around it. As I mentioned, UIAppearance properties get applied to all new instances of the particular class. But anything that works on those same properties after that, will override the UIAppearance values. And while this is definitely the right behavior, it is not always what you want to happen. This often leads to obscure bugs. So at the start, you will find yourself juggling between UIAppearance, Interface builder and code to figure out why something is not properly applied.

Another common issue you will encounter, is applying a rule too liberally. Sometimes you think a property is universal and should be applied everywhere, but in reality in turns out there are exceptions. Your first instinct might be to override that value after UIAppearances are applied, but that's also not ideal. It's just too implicit and hard to maintain. Luckily, this is a place where subclassing can help!

#### Subclassing and UIAppearance

I previously "preached" using subclassing with caution, and yet here we have a whole section dedicated to that. What makes UIAppearance different?

Truth be told, my previous statement still holds - you should still avoid subclassing whenever possible. However, it can still be quite helpful when using UIAppearance. Lets return to the small example above:

```swift
UILabel.appearance().textColor = themeColor
```

It seems harmless, but I would argue that `UILabel` is often a terrible candidate for UIAppearance. Even if you want to apply an overall theme to your application, there are usually labels that should have a different style. Sometimes, elements around those labels need to be branded, but the text should remain the same. For instance, you might want to brand a navigation bar in red, but the custom navigation title inside should be white for contrast.

This poses a question. How do you handle UI elements that don't have a single style, but several?

Since UIAppearance uses classes to differentiate between elements, a possible solution would be to make those elements instances of different classes. Getting back to the example, it would change to something like this:

```swift
BrandedLabel.appearance().textColor = themeColor
```

`BrandedLabel` would be an `UILabel` subclass that is actually quite empty. It needs to behave exactly as a normal label, but it... kind of need to be **not** `UILabel`. After that, you only need to make sure all labels that need to change are `BrandedLabel` instances.

It's not super clean, but gets the job done with not a lot of added complexity. Empty subclasses are not a really nice architecture, and I'm sure they violate some OOP principles. However, they hardly make the code harder to read, especially after you understand their purpose and learn to ignore them.

## Conclusion

App Branding is an inherently difficult problem to solve. There's no perfect solution and it's almost always a pain. I cannot claim that the techniques we discussed here are the right one. Definitely they are not an exhaustive list of solutions.

They are just a collection of items I found to be useful and have helped me a lot on the past. At the end of the day, every project is different and calls for different strategies. So you will have to do a bit of experimenting. Good luck!
