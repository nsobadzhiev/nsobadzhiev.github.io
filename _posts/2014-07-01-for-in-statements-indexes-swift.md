---
id: 241
title: For-in statements with indexes in Swift
date: 2014-07-01T14:25:14+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=241
permalink: /swift/for-in-statements-indexes-swift/
categories:
  - Swift
tags:
  - array
  - enumerate
  - for
  - for statement
  - for-in
  - ios
  - Swift
---
# A common problem with for-in statements

For statements are great! There are little programming languages that don't include a `for` of some sort. In it's natural form it looks something like this:

	for (int i = 0; i &lt; 7; i++)
	{
		// do something seven times
	}
	
However, iOS developers (as well as Perl, Python, Ruby, Bash and many others) are most likely seeing it in a more convenient form:

	for (NSObject* someObject in array)
	{
		// do something with this object
	}
	
This short version of the statement is just some synthax sugar that makes it easier to iterate over an array. It keeps developers from having to take care of array indexes. We use it so much that I&#039;m starting to forget the original form.

That being said, sometimes this for-in statement falls short. Consider this scenario. You have an array full of the names of participants in an event. You want to iterate over all of them, but also need the given participant&#039;s position in the collection (it&#039;s index). This means that each time the body of the loop is reached, we are going to need the name and the index at which it appears in the original array. This is hard to do with a `for-in`. We have to use the basic form of `for`.

Swift to the rescue!

---

**EDIT:**
As it turns out, there IS a way to achieve this in Objective-C. Thanks to Jurgis for acknowledging this in the comment section. His suggestion was to use the following code:

	[array enumerateObjectsUsingBlock:^(id object, NSUInteger idx, BOOL *stop) {
		// do something with object
	}];
	
It's not as nice (synthactically) as the one you will see shortly, but it does work and with a good comment beside it, none of your collegues shouldn't have any problem understanding it.
Jurgis, thanks again for your comment!

---

# For-in statements with indexes in Swift

Unfortunately, I don&#039;t think this feature has a name, so I&#039;ll just refer to it as for-in statements with indexes in Swift. It is a way to use Swift&#039;s common for-in construct, but also get an index everytime the loop&#039;s body is invoked. Using the example from the previous paragraph, it would look something like this:

	for (index, participant) in enumerate(participants) {
		// index is the index within the array
		// participant is the real object contained in the array
	}

Let&#039;s see what happened here:

* The `enumerate` function is used in order to iterate through the array, but instead of just returing the next object, everytime it need to provide an item, it returns a tuple containing both the object and an integer containing the index at which that object was encountered.
* Instead of using a single variable in order to control the loop, we have a whole tuple that will be filled with the data provided by the `enumerate` function above.
* Everytime the we enter the loop&#039;s body, we have both the object **and** it&#039;s index to work with.

This is another exmaple of how Swift improves on the experience given to developers by Objective-C. It was a common problem for programmers to need an index while they iterate a collection, and for-in statements with indexes in Swift resolved that issue. This allows people to not only write less code, but also make it a lot more readable.

You can use the comment section below to discuss whether or not you have always wished you could easily have an index whenever you are working with the for-in loop and will you be using that new feature in Swift. Thanks for reading!