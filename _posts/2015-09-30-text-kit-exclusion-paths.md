---
id: 391
title: Text Kit exclusion paths
date: 2015-09-30T13:07:29+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=391
permalink: /ios/text-kit-exclusion-paths/
categories:
  - iOS
  - Tutorials
tags:
  - exclusion path
  - ios
  - Objective-C
  - TextKit
  - UITextView
---
Normally, a good author would wait until the end to reveal his ultimate thoughts on a subject, but since I'm probably not a good author, I'll allow myself to skip right to it:

> What a time to be alive...!

That was my reaction when I first read about Text Kit exclusion paths. Seriously speaking,
I was generally impressed at the little gems that are now available within the Cocoa framework. Anyway, before I get too emotional, lets get started:

## What is Text Kit

Text Kit is a framework, introduced in iOS 7, that brings some advancements in text rendering for iOS projects.
To explain why this is so exciding, I'll have to take you back to the old days of iOS development.

Prior to iOS 6, while most UI components would provide a fair amount of text visiolization customizations, they were all limited to fonts, colors and alignments.
Even simple things like bolding a single word in a label was unavailable.

_Try explaining that to your designer..._

Naturally, this is a weird limitation since it's such a basic requirement. It's everywhere, so it's a bit difficult to convince people why this would be more difficult than they expect.

To be fair, it's not just iOS. Most platforms are not much better. If you want to perform even the most basic text formating operations, you have to go for HTML.

And that's exactly what happened in iOS 6. Suddently, "rich-text" formating was made nice and easy. But looking under the hood, one would quickly realize that what powers that new functionality is actually something you should be familiar with, namely WebKit. So in reality, that was no better than using HTML.

To be honest, a little HTML processing has never hurt anyone (actually, it probably has), and I'm pretty sure there wouldn't be any performance considerations unless you **really** go overboard with it. However, there's something inside me that cringes everytime I hear about using HTML in a native app.

Luckily, this is no longer an issue. Because with the "invention" of iOS7, Text Kit has got your back.

## What Text Kit can do for you

Text Kit will not pay the rent when it is due, unfortunately, but it will make you **not** have a panic attack everytime you have to visualize text in anything but the most trivial way.

If you go to the [reference page for Text Kit][TextKit reference] you'll find all kinds of low level text and font... thingies like glyphs. As a man who can hardly distinguish between fonts, I wouldn't want to go into details about this, though. I'll just stick to the other aspects of Text Kit.

### Changing text layout

So, what even I can find useful in a framework such as Text Kit, is its ability to get you involved into the way text is being laid out on screen.

A certain limitation I've stumbled upon several times in my career, is that all text containers in the Cocoa framework (and most others), are constrainted to a rectangular shape. I don't really blame the architects, since that covers 99% percent of the cases. But what if you wanted to have text floating around an image like in a magazine? Pretty cool, eh? WELL, YOU CAN'T HAVE IT!

Why? Well, you're faced with the following problem...
All UI components that display text have a rectangular shape. And also, they don't expose that much functionality to allow you to affect the way text is being layed out. Now that's tough... You'll either have to ditch your floating text idea, or go for Core Text or something. And suddenly, you're working weekends, you let yourself go and you realize you no longer have any friends (provided that you had any before).

I might have over-exaggerated that last part, but you get the point. It's useful to add additional constraints on the test layout.

### Text Kit exclusion paths

Text Kit exclusion paths to the rescue!
Here's the statement you've all been waiting for while I keep writing nonsense:

Text Kit exclusion paths allow you add additional rules affecting how text is rendered inside a text view. Using them, you can specify locations within the container where text shouldn't be drawn. This effectively means that achieving a "magazine-like" UI, where text floats around images, becomes rather easy. You just add the frames of your image views as exclusion paths and (hoprefully) your wishes come true.

Without further adu... some code:

> UIBezierPath* path = [UIBezierPath bezierPathWithRoundedRect:self.imageView.frame
                                               byRoundingCorners:UIRectCornerAllCorners
                                                     cornerRadii:CGSizeMake(15, 15)];
  self.textView.textContainer.exclusionPaths = @[path];

Only two lines of code... iOS development is getting lazy. So first of all, exclusion paths are always bezier paths.
I realize some of you are ready to close the tab, but in this case... SHAME ON YOU. It's not that bad. Also, the `UIBezierPath` class has a lot of simple constructors specially tailored to those, unfamiliar with graphics.

In our case, we don't really need a complex curve, so we can just get away with creating a bezier curve based on a rectangular. And we will also include a corner radius to make the text frame more interesting.

Once that done, we set the `exclusionPaths` property with an array containing all curves we created. Really simple...

### Text Kit rectangular for text

Maybe I'm being cheeky, but I'm wondering if Text Kit can do more. What's another text related problem Cocoa is notorious for NOT solving.
OH, got one! And I'm sure you've also encountered it.

#### How much screen space will a text view need to draw a certain amount of text?

I don't know about you, but I've had a lot of sleepeless night because of this. Sure, Autolayout resolves this problem most of the time, but not always.
Also there's this method:

> [myLabel.text sizeWithFont:[UIFont systemFontOfSize:14]];		// depricated btw

But last time I chacked it didn't work very well for multiline labels.
So it is possible... no, I doubt it. But maybe... just maybe... it works with Text Kit...?

> CGRect usedRect = [self.textView.layoutManager usedRectForTextContainer:self.textView.textContainer];

IMPOSSIBRU! As I said in the beginning... **What a time to be alive!**

### Text Kit NSTextContainer and NSLayoutManager

Now that you're totally impressed by the Text Kit exclusion paths, lets get to the boring part. What were those objects we used in the previous examples, namely `layoutManager` and `textContainer`. As seen on the [reference page for Text Kit][TextKit reference], the architecture for the Text Kit framework is as follows:

![Text Kit classes](https://developer.apple.com/library/ios/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/Art/textkitarchitecture_2x.png)

Basically there are three classes to consider:
* NSTextContainer
* NSLayoutManager
* NSTextStorage

#### NSTextContainer

Represents a region on screen where text should be drawn. And as we saw previously, contains exclusion paths for areas on screen that should not have text.

#### NSLayoutManager

NSLayoutManager is responsible for converting the text you provide into a series of letters (glyphs within the context of Text Kit) drawn on screen

#### NSTextStorage

NSTextStorage might sound fancy, but it's actualy the string object that stores the text you want displayed.

Feel free to go ahead and read about each of these classes and discover all goodies Text Kit can offer.

[TextKit reference]: https://developer.apple.com/library/ios/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/CustomTextProcessing/CustomTextProcessing.html#//apple_ref/doc/uid/TP40009542-CH4-SW1