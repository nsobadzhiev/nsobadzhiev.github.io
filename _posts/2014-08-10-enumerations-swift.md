---
id: 263
title: Enumerations in Swift
date: 2014-08-10T14:18:26+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=263
permalink: /swift/enumerations-swift/
categories:
  - Swift
  - Swift vs Objective-C
tags:
  - associated values
  - enum
  - enumerations
  - ios
  - raw values
  - Swift
---
_This post is part of a [Swift vs Objective-C series][Swift vs Objective-C index] where we look at new features in Swift and how they compare to their predecesors from Objective-C.
In this article, we are going to discuss enumerations in Swift._

# What's new with enumerations in Swift

Another day, another programming language...
With the addition of Swift as a development platform for writing Mac and iOS apps, it is time to think about translating our knowledge of Objective-C to Swift. Enumerations are a perfect place to revisit as Swift contains a lot of improvements and new features as opposed to it's predecessor.

For the most part, you can use enumerations in Swift in the same way you did previously in Objective-C... or C/C++ for that matter. There are some minor syntax changes, but other than that the mechanics are the same.

Let's look at an example. Let's say we are developing a reader application that displays different types of books - PDFs, EPUBs, web pages etc. We need a variable that determines which type of media we are currently showing to the user.

In Objective-C we would create an `enum` similar to this:

	enum eBookType
	{
		eBookTypePdf,
		eBookTypeEpub,
		eBookTypeWebPage,
		eBookTypeMsWord,
	};
	
Now if we want to translate this to enumerations in Swift, we'd write something like:

	enum BookType {
    	case Pdf
    	case Epub
    	case WebPage
    	case MsWord
	}
	
As you can see, there are a few changes.
First of all, each entry is prefixed with the `case` keyword. It tells the compiler that a new line of member definitions starts. But you can also put several definitions on a single line:

	enum BookType {
		case Pdf, Epub, WebPage, MsWord
	}
	
You might have noticed that the naming convention has changed from the Objective-C to the Swift example. Though most things are not covered by the language's specification and are largely subjective depending on your coding style, I'll try to explain my motivation for the choice of names.

* Enumeration name

 Since enumerations in Swift are more similar to classes and structures than their C counterparts, their names should resemble class names. That's why the `BookType` name starts with a capital letter. Even though this is not mandatory, it is mentioned in Apple's book as the recommended style.
 
* Member names

 I like to put the enum name as the begining of the member. That's how I always know which data type it belongs to. However, enumerations in Swift use the dot notation to access members. So, to use the `Pdf` member, you would write `BookType.Pdf`. In such case, it doesn't make sense to use the naming convention from the Objective-C example. Otherwise, the finished statement looks awkward: `BookType.BookTypePdf`.
 
# New features

So far, enumerations in Swift do not look that different. However, we have only scratched the surface. While, they support all the functionality from C enums (with little modification), enumerations in Swift also introduce a number of new concepts. Here they are:

## Dot notation

I already mentioned this above, but let's repeat it nevertheless. Similar to structures, enumerations in Swift use the dot notation in order to specify a member:

	enumeration_name.member_name
	
This makes statements more explicit about the enumeration they are refering to. It may not seem that important, but I feel that this is a real upgrade from the way C enums work. Unless you enforce a naming convention that helps you link members to their respective enumerations, it might become difficult to know which data type a member belongs to.

## Enumeration member values

In C, each enum member is assigned an integer number. Getting back to our previous example, the members of `eBookType` have the following values:

* eBookTypePdf - 0
* eBookTypeEpub - 1
* eBookTypeWebPage - 2
* eBookTypeMsWord - 3

Whenever you refer to `eBookTypePdf`, for instance, it is the same as using `0`.
This does not apply to enumarations in Swift. Instead, each Swift enum member has a fully-fledged value that could be either a string, a character or a number. This allows you to "attach" additional information to an enumeration member. This additional information is called an **associated value**

### Associated values

Associated values allow you to add a string, character or number variable to an enumeration member. Let's illustrate this with an example. Using the enum from the previous paragraph

	enum BookType {
		case Pdf
		case Epub
		case WebPage
		case MsWord
	}

we would also want to link each book with it's title. The `Epub` member specifies that the current book is in Epub file format and it's associated value indicates it's name. So, when writing a view controller displaying the book, the following statement might prove useful:

	viewController.book = BookType.Epub("Bedtime stories")
	
Naturally, in order to support this string value, we need to update our enum definition:

	enum BookType {
		case Pdf(String)
		case Epub(String)
		case WebPage(String, NSURL)
		case MsWord(String)
	}
	
Notice how `WebPage` has two associated values. Enumerations in Swift are not limited to a single associated value.

### Switch statements using enumerations in Swift

Using enums in switch statements is not very different from before, but there are several things to note.
First of all, you have to provide a `case` for all members. If you don't want to do that, you need to add a `default` case to handle all unspecified members:

	let book = BookType.MsWord
	switch book {
	case .Pdf:
    	println("This is a PDF")
	case .MsWord:
		println("This is a Word file")
	default:
    	println("Either an EPUB or a web page")
	}
	
In this switch, we are only explicitly handling the PDF and MSWord cases. Since Web page and EPUB are not, we had to provide a default case for them. If we didn't, the compiler would have returned an error.

You can also integrate associated values into your switch statement:

	let book = BookType.MsWord("Enumerations in Swift for dummies")
	switch book {
	case .Pdf(let bookTitle):
		println("This is a PDF: \(bookTitle)")
	case .MsWord(let bookTitle):
		println("This is a Word file: \(bookTitle)")
	default:
		println("Either an EPUB or a web page")
}

## Raw values

Raw values are similar to the integer values enum members have in C/C++. They are the default values a member of enumerations in Swift has. It always has the same data type across all members and like associated values can be either a string, a number or a character. 

Here's how we can assign numbers to the values in our `BookType` enumeration:

	enum BookType: Int {
		case Pdf = 0
		case Epub = 1
		case WebPage = 2
		case MsWord = 3
	}

Integer raw values **do** auto-increment if not explicitly specified, but you cannot assign the same raw value twice within a single enumeration in Swift.

You can always access an enum member's raw value via the `toRaw` method. The opposite is also possble using the `fromRaw` function:

	let rawPdf = BookType.Pdf.toRaw()
	let book = BookType.fromRaw(3)		// MsWord
	
In this example the `book` variable is of type optional BookType (`BookType?`), because it might return `nil` if you provide a raw value that does not exist.

[Swift vs Objective-C index]: http://devmonologue.com/ios/swift/swift-vs-objective-c/