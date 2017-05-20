---
id: 103
title: Regular Expressions in iOS
date: 2014-02-09T12:26:14+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=103
permalink: /ios/regular-expressions-in-ios/
categories:
  - iOS
tags:
  - ios
  - iPhone
  - NSRegularExpression
  - ObjC
  - Objective-C
  - regex
  - regular expressions
---
Let's take a moment to talk about regular expressions, or regex as the cool kids call it. As a software developer you probably already know about them, or at least you should. Either way, we will start with a short overview on what they are and how they are used, and after that we will go into specifics of using NSRegularExpression – the Cocoa class that brings regex support to iOS.

<span style="text-decoration: underline;"><b>What are regular expressions?</b></span>

Regular expressions are a whole subsection of mathematics. There is a very long and very extensive theory describing what they are and what properties they have. Luckily enough, I am not going to go into details on that mainly because I don't want to bore you to death with the second paragraph of my post. Also, for your daily programming needs, you will not need to know all that – it should be enough to know roughly what they are and how to use them. Intuitively, regular expressions represent a search pattern that matches certain portions of text. In fact, when you enter a word in any search bar (on any device), you are effectively typing in a very simple regular expression. You specify a keyword and it matches every location in the text (a web page for instance) where that particular sequence of letters is encountered. That's essentially what regex does, however it also allows you to construct a lot more complex patterns, like “All lines containing less than 80 characters” or “All words in quotes”. This ability is very powerful even though you might not realize it at first. System administrators use it to list specific files or to filter logs for relevant information. It programming, they can be used for complex text processing and analysis. The sky's the limit with regular expressions.

<span style="text-decoration: underline;"><b>How to construct regular expressions?</b></span>

In this section we are going to discuss how to create regular expression statements. For now, we are not going to go into Objective-C specifics. What you read here, can be applied for most programming languages – most of them share the same rules and you should be able to copy-paste the same patterns and expect the same results no matter if you use Bash, Perl or Objective-C.

For the most part, regular expression statements seem very intimidating initially with their strange syntax, but once you get used to it, they are not that bad.

As I mentioned, a regular expression is a string that contains a pattern that will be used to match portions of the text it's used against. The string is a combination of “literal” characters and special ones that represent any text with a certain properties. You can think of the matching process as a series of substitutions that, if successful, determine that a certain substring can be included in the results. For instance, the “.” character will match any character. If used in a regex like “te.t”, the framework will start looking for sections in the form of “te_t” and will allow the third character to match anything. Results might be words like “test”, “text” and “teet”. However, note that the word “tet” will NOT match, because “.” specifies that it doesn't matter which character's there, as long as there is something. If you want to include “tet” in the results you have to use another piece of regex magic. But more on that later. Now, let see a few more options:

<span style="text-decoration: underline;"><b>\:</b></span>

If you want to use a regex symbol literally (without it's special meaning – like the “.” in the previous example), you can prefix it with “\” to “escape” it.

<span style="text-decoration: underline;">“<b>.”:</b></span>

Matches any character, but as already discussed, does not match if there is no character at all.

<span style="text-decoration: underline;">“<b>^”:</b></span>

Matches the beginning of the line. In many implementations, regular expressions are matched on a “per line” basis – each line is processed separately. It is important to have that in mind.

<span style="text-decoration: underline;">“<b>$”:</b></span>

Matches the end of the line (the \n character).

<span style="text-decoration: underline;">“{n,m}”:</span>

Indicates that the character or group to the immediate left can be matched <b>n</b> to <b>m</b> times. For instance, “a{2,3}” will match “aa” and “aaa”. “a{2,}” can also be used to indicate “two or more”.

<span style="text-decoration: underline;">“<b>|”:</b></span>

Or.

<span style="text-decoration: underline;">“<b>*”:</b></span>

Matches the character or group to the immediate left zero or more times. Note that, by default, regular expressions are greedy – they will match as many characters as possible – they wont stop as soon as a match is found.

<span style="text-decoration: underline;">“<b>+”:</b></span>

Matches the character or group to the immediate left one or more times. Note that, by default, regular expressions are greedy – they will match as many characters as possible.

<span style="text-decoration: underline;">“<b>?”:</b></span>

Matches the character or group to the immediate left one or zero times. Again, because regular expressions are greedy, “?” will match one, instead of zero times if both are available.

<span style="text-decoration: underline;">“<b>\A”:</b></span>

Matches the beginning of the input text.

<span style="text-decoration: underline;">“<b>\z”:</b></span>

Matches the end of the input text.

<span style="text-decoration: underline;">“<b>(...)”:</b></span>

Creates a group with the expression inside the parentheses. It is used in conjunction with operators like “*” and “?” so that they apply for more than one character.

<span style="text-decoration: underline;">“<b>(?!...)”:</b></span>

Matches only if the expression inside the parentheses does not.

<span style="text-decoration: underline;">“<b>[...]”:</b></span>

Matches any character in the set (any character that is between the brackets).

<span style="text-decoration: underline;">“<b>[^...]”:</b></span>

Matches all characters that are <b>not </b>in the brackets.

There are of course, other operators, but let's not get carried away (I'll leave them as homework :) ). The above mentioned is more than enough to get you started and if you someday need anything else, you can always look it up.

With the boring stuff out of the way, let's get started with some examples.

<span style="text-decoration: underline;"><b>Finding all single line comment in source code:</b></span>

This example represents one of the simpler tasks you can accomplish using regular expressions – matching everything, provided that in contains several exact characters. More specifically, in this example we will need to match all character on a given line, starting with a “//” sequence. Any ideas...? Anyway, this is the solution:

“//.*”

So, how does that work? First of all we explicitly say that we want the results to begin with “//”. After that the “.*” expression specifies that we want to include everything until the end of the line. Remember that “.” will match any character and the “*” will match the character to the left zero or more times. That means any character after the “//” sequence will be included. Since regex is greedy, the matching will not stop until it encounters a character that no longer satisfies the pattern. Also, as mentioned, by default, it will process each line separately, so each result will be restricted to the closest “\n” character.

<span style="text-decoration: underline;"><b>Finding all instances of the word “Obj-C” or “Objective-C”:</b></span>

In this example, we are going to try using the logical OR in order to match two different patterns at the same time. This is fairly straight-forward and you should be able to figure it out for yourself, but here it is:

“(Obj-C|Objective-C)”

The “|” character here indicates that a string has to match either operand in order to match the pattern. The parentheses are not mandatory here but most of the time they are used because they would be a part of a bigger expression and will specify the end of the “or” statement.

<span style="text-decoration: underline;"><b>Finding opening XML tags:</b></span>

We will have to match all <i>substrings starting with “&lt;” and ending with “&gt;”, containing a tag name and series of attributes</i>. So the following should be enough, right?

“&lt;.+&gt;”

That was easy, wasn't it? Well... no... What will happen to a string like:

“&lt;tag1 attr1=”true” &lt;tag2/&gt;&gt;”

It certainly does not look like an XML tag, yet, our regular expression is going to match it. OK, let's try again. An XML tag is a <i>substring that starts with “&lt;” and ends with “&gt;” and does </i><b><i>not</i></b><i> contain other instances of “&lt;” and “&gt;”. </i>That's a little more complicated, isn't it? Truth is that many real world problems require just that. So let's resolve the issue. We need to add an additional restriction in our pattern. We have to accept all character sequences that do not include other tags. Now scroll up a little and read the “(?!...)” section once again. It allows us to specify a pattern that we don't want present in the final result. Using it, we complete the task by using the following pattern:

“&lt;((?!&lt;).)*&gt;”

That's better! The day is sunny once again and... wait! What if I had to match the following string:

“&lt;tag1 attr=”chicken”&gt; some text &lt;tag2 enabled=”false”&gt;”

It matches the whole string. But why??? Well, as I hinted multiple times, regular expressions are greedy. They will match as much text as possible. So when processing reaches “&lt;tag1 attr=”chicken”&gt;” the pattern matches, however, the algorithm continues adding characters to the result until it reaches a position after which the text no longer conforms to the regular expression. This this case, it is the last character and that's why the whole string matches the pattern. We need to figure out a way to stop the regex after the first closing symbol. To do that, we will use yet another piece of magic - “*?”. Intuitively, this says “Match the preceding expression zero or more times, but prefer less matches”. Non-intuitively, the “*” will match any number of times, but the “?” (remember that it means “one of zero times”) will make it stop as soon as the pattern successfully matches.

So let's see what we end up with:

“&lt;((?!&lt;).)*?&gt;”

But wait! There's more! Although, this will work fine for the most part, there is a little detail we need to consider. I already mentioned, regular expressions are usually processed on a per-line basis. This means that each line in the input is handled separately. So if we had an XML with several lines like this:

“&lt;tag1 enabled=”true”

required=”false”/&gt; Hello &lt;tag2 /&gt;”

The regex will return only one result - “&lt;tag2 /&gt;”, because it doesn't test the whole string at once (unless you tell it to, but more on that later). So if this is important to you, you will have to modify your expression. Let's see how – I promise there will be no more surprises:

“(^((?!(&lt;|&gt;)).)*&gt;|&lt;((?!&lt;).)*?(&gt;|$))”

Wow! That escalated quickly. Let me explain. What's new here is that we attempt to handle situations where the closing/opening tag character is not on the same line. That means text from the beginning of the line to the first closing symbol (provided that it does not contain “&lt;”) as well as text for the last “&lt;” to the end of the line if a closing “&gt;” is missing. You should already know that “^” matches the beginning of a line and “$” matches the end of it. The rest is nothing new as a concept – you should be able to figure it out...eventually.

You might have noticed that the last expression handles a tag with two lines, but what about three... or four? But let's not get carried away.

So what seemed like an easy task, turned out to be a lot harder than expected. We started with a simple expression with four characters and ended up with a huge pattern that might give you a headache just looking at it. Truth is, in my experience, it is not uncommon thing to happen while using regex. Not that I want to discourage you, I just want you to be prepared, because regular expressions are coming for you... and there is nothing you can do to stop them! Sooner or later every programmer has to face them.

<span style="text-decoration: underline;"><b>Regular expressions in iOS (NSRegularExpression)</b></span>

<i>Before I begin, I would like to share a bit of personal advice. When you start using regular expressions in your source code, in my experience, debugging is not enough. You have to be sure your patterns are correct before you start using them in your program. There is no easy way to determine what's wrong in the matching process so it would be difficult to find errors by conventional methods. I would recommend that you fire up your favorite text editor and test your expressions there – it's a lot more visual and you can quickly see what the problem is. I personally use <a href="http://macromates.com">TextMate</a>, but you can probably use whatever editor you currently have. Unfortunately, the built-in TextEdit application does not support regex, but Xcode does.</i>

[caption id="attachment_104" align="aligncenter" width="300"]<a href="{{ site.url }}/images/2014/02/regexTextMate.png"><img class="size-medium wp-image-104" alt="Figure 2: Regular Expressions in TextMate" src="{{ site.url }}/images/2014/02/regexTextMate-300x159.png" width="300" height="159" /></a> <em>Figure 2: Regular Expressions in TextMate</em>[/caption]

<em>Just press CMD + F to go the “find” menu, click on the magnifying glass, choose “Edit find options...” and select “Regular Expression” as “Matching Style”.You can even use grep if you are not afraid of the terminal.</em>

[caption id="attachment_105" align="aligncenter" width="300"]<a href="{{ site.url }}/images/2014/02/regexXcode.png"><img class="size-medium wp-image-105" alt="Figure 1: Regular Expressions in Xcode" src="{{ site.url }}/images/2014/02/regexXcode-300x157.png" width="300" height="157" /></a> Figure 1: Regular Expressions in Xcode[/caption]

<span style="line-height: 1.5em;"> </span>

Now that we covered some regex basics, it is time to dig into Cocoa's implementation – the <a href="https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSRegularExpression_Class/Reference/Reference.html">NSRegularExpression</a> class. Let's start with an example and afterwards I will explain what everything does:
<pre class="brush:objc">NSError* regexError = nil;
NSRegularExpression* regex = [NSRegularExpressionregularExpressionWithPattern:@"&lt;((?!&lt;).)*?&gt;"
options:NSRegularExpressionCaseInsensitive|NSRegularExpressionDotMatchesLineSeparators
  error:&amp;regexError];

if (regexError)
{
    NSLog(@"Regex creation failed with error: %@", [regexError description]);
    return nil;
}

NSArray* matches = [regex matchesInString:self.htmlString 
                                  options:NSMatchingWithoutAnchoringBounds 
                                    range:NSMakeRange(0, self.htmlString.length)];</pre>
<span style="font-family: 'Times New Roman', serif;"><span style="font-size: medium;"><span style="color: #000000;">So, the code above will apply the regular expression we discussed in the previous section, using Objective-C. You will see that since all the heavy-lifting of creating the pattern is done, applying it is relatively simple. Not surprisingly, first you create an instance of the NSRegularExpression class and after that you apply it to an NSString. There not much more to it than that.</span></span></span>

<span style="font-family: 'Times New Roman', serif;"><span style="font-size: medium;"><span style="color: #000000;">Let's see how are NSRegularExpression created. In the example above, we use a class method in order to get an autoreleased (if you're not using ARC) regular expression object. You have to supply a string, containing the pattern. As far as I know, there is no other way to supply the regex to the object (inlike NSPredicate where there is way to specify conditions without hardcoding a string). With the seco</span><span style="color: #000000;">n</span><span style="color: #000000;">d parameter, you will be able to set some parameters that will apply to the search. In this case, we want our matching process to ignore character cases so we are using </span><span style="color: #2e0d6e;"><span style="font-family: Menlo-Regular, serif;">NSRegularExpressionCaseInsensitive</span></span><span style="color: #2e0d6e;">. </span><span style="color: #000000;">This is the standard option you are most likely to see in tutorials and is probably enough for your day-to-day regex needs. However, in our case, we are going to add another option – </span><span style="color: #2e0d6e;"><span style="font-family: Menlo-Regular, serif;">NSRegularExpressionDotMatchesLineSeparators</span></span><span style="color: #000000;">. </span><span style="color: #000000;">Please note that in the code snippet, we didn't use the final version of the regular expression from the previous section. We used the one before that – the one that didn't account for XML tags spanning on two lines. The problem there was that regular expression frameworks by default were matching each line separately, so they couldn't always handle results that contain new line characters. However, by adding the </span><span style="color: #2e0d6e;"><span style="font-family: Menlo-Regular, serif;">NSRegularExpressionDotMatchesLineSeparators</span></span><span style="color: #000000;"> option in NSRegularExpression, we force it to continue matching even after it reaches the end of the line.</span></span></span>

<span style="font-family: 'Times New Roman', serif;"><span style="font-size: medium;"><i><b><span style="color: #000000;">Note: </span></b><span style="color: #000000;">Always have the fact that regular expression are matched on a per-line basis in mind, because it might cause frustration when a piece of text does not match, especially in larger input texts. When you test your code, you are more likely to use shorter examples where everything is fine, but as soon as your code encounters some real world problems, it starts failing for no reason. Consider yourself warned!</span></i></span></span>

<span style="font-family: 'Times New Roman', serif;"><span style="font-size: medium;"><span style="color: #000000;">The last parameter that is required in order to create an NSRegularExpression can be used to obtain a reference to an error that occurred while creating the regex. Now, I don't see how your users will react to an “Unable to create regex” error, but at the very least, you can log the error in the console for troubleshooting.</span></span></span>

<span style="font-family: 'Times New Roman', serif;"><span style="font-size: medium;"><span style="color: #000000;">With the regular expression created, it is time to apply it to some text. In this example, we obtain an array </span><span style="color: #000000;">containing</span><span style="color: #000000;"> all results, but the API provides a whole range of method to fit your needs – finding matches, number of matches, first match, replacing matches... you name it. You can find the complete list on the <a href="https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSRegularExpression_Class/Reference/Reference.html">NSRegularExpression reference page</a>.</span></span></span>

<span style="font-family: 'Times New Roman', serif;"><span style="font-size: medium;"><span style="color: #000000;">The rest is fairly simple. You supply the text you want to use as input, specify the range of the string you want to restrict the matching to as well as some additional settings (explained <a href="https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSRegularExpression_Class/Reference/Reference.html#//apple_ref/c/tdef/NSMatchingOptions">here</a>).</span></span></span>

<span style="text-decoration: underline;"><b>Conclusion</b></span>

It's been a long post, but we (or at least some of us) finally made it. Regular expressions in Objective-C. You might feel intimidated by them and I don't blame you. But understanding them takes time. And it certainly takes more than one blog post. That's the reason I wrote it in the first place. Personally, I cannot remember how many I've read and I still have a long way to go. It is always good to learn about everybody's take on regular expressions. And this is mine... I hope you enjoyed this article and found it useful.

Thanks for reading!

	<link href="http://purl.org/dc/elements/1.1/" rel="schema.DC" hreflang="en" /> 	<link href="http://purl.org/dc/terms/" rel="schema.DCTERMS" hreflang="en" /> 	<link href="http://purl.org/dc/dcmitype/" rel="schema.DCTYPE" hreflang="en" /> 	<link href="http://purl.org/dc/dcam/" rel="schema.DCAM" hreflang="en" />

<style type="text/css"><!--
@page {  }
table { border-collapse:collapse; border-spacing:0; empty-cells:show }
td, th { vertical-align:top; font-size:12pt;}
h1, h2, h3, h4, h5, h6 { clear:both }
ol, ul { margin:0; padding:0;}
li { list-style: none; margin:0; padding:0;}
<!-- "li span.odfLiEnd" - IE 7 issue-->
li span. { clear: both; line-height:0; width:0; height:0; margin:0; padding:0; }
span.footnodeNumber { padding-right:1em; }
span.annotation_style_by_filter { font-size:95%; font-family:Arial; background-color:#fff000;  margin:0; border:0; padding:0;  }
* { margin:0;}
.fr1 { border-style:none; font-size:12pt; margin-bottom:0in; margin-left:0in; margin-right:0in; margin-top:0in; padding:0in; font-family:Times New Roman; vertical-align:top; writing-mode:lr-tb; }
.fr2 { border-style:none; font-size:12pt; margin-bottom:0in; margin-left:0in; margin-right:0in; margin-top:0in; padding:0in; font-family:Times New Roman; text-align:left; vertical-align:top; writing-mode:lr-tb; }
.fr3 { font-size:12pt; font-family:Times New Roman; text-align:center; vertical-align:top; writing-mode:lr-tb; margin-left:0in; margin-right:0in; margin-top:0in; margin-bottom:0in; padding:0in; border-style:none; }
.fr4 { font-size:12pt; font-family:Times New Roman; vertical-align:top; writing-mode:lr-tb; margin-left:0in; margin-right:0in; margin-top:0in; margin-bottom:0in; padding:0in; border-style:none; }
.Caption { font-size:12pt; font-family:Times New Roman; writing-mode:page; margin-top:0.0835in; margin-bottom:0.0835in; font-style:italic; }
.P1 { font-size:12pt; font-family:Times New Roman; writing-mode:page; }
.P10 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:none ! important; font-weight:normal; }
.P11 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:none ! important; font-weight:normal; }
.P12 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:none ! important; font-weight:normal; }
.P13 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:none ! important; }
.P16 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:none ! important; font-weight:normal; }
.P18 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P19 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P2 { font-size:12pt; font-family:Times New Roman; writing-mode:page; }
.P20 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P21 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P22 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P23 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P24 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P25 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P26 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P27 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P28 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P29 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P3 { font-size:12pt; font-family:Times New Roman; writing-mode:page; }
.P30 { font-size:12pt; font-family:Times New Roman; writing-mode:page; }
.P31 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:italic; text-decoration:underline; font-weight:bold; }
.P32 { font-size:12pt; font-family:Times New Roman; writing-mode:page; }
.P33 { font-size:12pt; font-family:Times New Roman; writing-mode:page; }
.P34 { font-size:12pt; font-family:Times New Roman; writing-mode:page; }
.P35 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:italic; text-decoration:none ! important; font-weight:bold; }
.P36 { font-size:12pt; font-family:Times New Roman; writing-mode:page; font-style:normal; text-decoration:none ! important; font-weight:bold; }
.P37 { font-size:12pt; font-style:italic; margin-bottom:0.0835in; margin-top:0.0835in; font-family:Times New Roman; writing-mode:page; }
.P4 { font-size:12pt; font-family:Times New Roman; writing-mode:page; }
.P5 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P6 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:underline; font-weight:bold; }
.P7 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:none ! important; }
.P8 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:none ! important; font-weight:normal; }
.P9 { font-size:12pt; font-family:Times New Roman; writing-mode:page; text-decoration:none ! important; font-weight:normal; }
.Bullet_20_Symbols { font-family:OpenSymbol; }
.T10 { text-decoration:none ! important; font-weight:normal; }
.T11 { text-decoration:none ! important; }
.T13 { font-weight:bold; }
.T14 { font-weight:bold; }
.T18 { font-style:italic; text-decoration:none ! important; font-weight:normal; }
.T19 { font-style:italic; text-decoration:none ! important; font-weight:normal; }
.T20 { font-style:italic; text-decoration:none ! important; }
.T21 { font-style:normal; }
.T22 { font-style:normal; text-decoration:none ! important; font-weight:normal; }
.T23 { font-style:normal; text-decoration:none ! important; font-weight:normal; }
.T24 { font-style:normal; text-decoration:none ! important; font-weight:normal; }
.T25 { font-style:normal; text-decoration:none ! important; font-weight:normal; }
.T26 { font-style:normal; text-decoration:none ! important; font-weight:normal; }
.T27 { color:#c41a16; font-family:Menlo-Regular; font-size:11pt; }
.T29 { color:#5c2699; font-family:Menlo-Regular; font-size:11pt; }
.T30 { color:#5c2699; font-family:Menlo-Regular; font-size:11pt; text-decoration:none ! important; font-weight:normal; }
.T31 { color:#5c2699; font-family:Menlo-Regular; font-size:11pt; text-decoration:none ! important; font-weight:normal; }
.T32 { color:#000000; }
.T33 { color:#000000; font-family:Menlo-Regular; font-size:11pt; }
.T34 { color:#000000; font-family:Menlo-Regular; font-size:11pt; }
.T35 { color:#000000; font-family:Menlo-Regular; font-size:11pt; text-decoration:none ! important; font-weight:normal; }
.T36 { color:#000000; font-family:Menlo-Regular; font-size:11pt; text-decoration:none ! important; font-weight:normal; }
.T37 { color:#000000; font-family:Menlo-Regular; font-size:11pt; text-decoration:none ! important; }
.T38 { color:#000000; font-family:Menlo-Regular; font-size:11pt; text-decoration:none ! important; font-weight:bold; }
.T39 { color:#000000; font-family:Menlo-Regular; font-size:11pt; text-decoration:none ! important; font-weight:bold; }
.T40 { color:#000000; }
.T41 { color:#000000; font-weight:normal; }
.T42 { color:#000000; font-weight:normal; }
.T43 { color:#000000; }
.T44 { color:#aa0d91; font-family:Menlo-Regular; font-size:11pt; }
.T45 { color:#aa0d91; font-family:Menlo-Regular; font-size:11pt; text-decoration:none ! important; }
.T46 { color:#aa0d91; font-family:Menlo-Regular; font-size:11pt; text-decoration:none ! important; font-weight:bold; }
.T47 { color:#2e0d6e; }
.T48 { color:#2e0d6e; font-family:Menlo-Regular; font-size:11pt; }
.T49 { color:#2e0d6e; font-family:Menlo-Regular; font-size:11pt; text-decoration:none ! important; font-weight:normal; }
.T5 { text-decoration:none ! important; }
.T50 { color:#3f6e74; font-family:Menlo-Regular; font-size:11pt; }
.T51 { color:#3f6e74; font-family:Menlo-Regular; font-size:11pt; text-decoration:none ! important; font-weight:bold; }
.T52 { color:#1c00cf; font-family:Menlo-Regular; font-size:11pt; }
.T6 { text-decoration:none ! important; font-weight:normal; }
.T7 { text-decoration:none ! important; font-weight:normal; }
.T8 { text-decoration:none ! important; font-weight:normal; }
.T9 { text-decoration:none ! important; font-weight:normal; }
<!-- ODF styles with no properties representable as CSS -->
.T1 .T12 .T15 .T16 .T17 .T2 .T28 .T3 .T4 .T53 .T54  { }
--></style>