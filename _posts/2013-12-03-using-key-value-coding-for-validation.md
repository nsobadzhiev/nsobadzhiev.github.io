---
id: 20
title: Using Key Value Coding validation
date: 2013-12-03T20:40:23+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=20
permalink: /ios/using-key-value-coding-for-validation/
categories:
  - error-handling
  - iOS
  - Key-Value Coding
  - KVC
  - validation
tags:
  - Key-Value
  - KVC
  - KVO
---
Key value coding in a powerful tool in Objective-C's Cocoa framework that allows you to change an NSObject subclasses's data without using properties and setters. It uses a mechanism, similar to the one used for adding data to an NSMutableDictionary. For instance, if I wanted to change a property named "brand" in a "Car" class, using properties, I would write something like:

<code>car.brand = @"Honda";</code>

Using Key Value Coding, or KVC,  I can achieve the same thing typing:

<code>[car setValue:@"Honda" forKey:@"brand"];</code>

At first glance, the latter might seam like it doesn't make much sense. First of all, it introduces some hardcoded strings, which is never a good thing. Not only that, hardcoding a property's name can be very dangerous if we later decide to change that property's name, or remove it altogether. If this happens, the application will crash with an exception, saying the Car in not key value compliant for key "brand".

Another disadvantage of using KVC is that it might seam like it decreases dependency in the code because you would only need to use NSObject (KVC is built into NSObject). In the example above, we don't need to expose the Car class, we are only setting a property in an object of a class, we don't need to know about. However, the fact that the header file for Car is not imported, doesn't mean that there is no dependency. We still have to be sure that the object is an instance of the Car class, or at the very least, make sure it has a property named "brand".

Despite this, Key value coding can be a powerful weapon if used appropriately.  It adds a new level of abstraction when dealing with different types of data classes. Normally, it would be hard to generalize some code to work for a wide range of data classes. Let's say you want to be able to dynamically map values to a data model by using plists. It would be inconvenient to put method names in it (it's possible, but it doesn't seam right - to me anyway). Using KVC, you will be able to put property names in that plist in order to do the mapping. And since you made the effort to implement reading those plists, you will most likely want to be able to use not only one type of data class, but a wide range of them. And since KVC is built into NSObject, you will have no problem doing so - all you need to know is the name of the property you need to change.

Another major advantage in using KVC is the ability to integrate validation and error handling right into your data model. The problem with error handing is that it's too boring and tedious and programmers tend to avoid it like the plague until the last stage of development. This means that often it is not considered during the design phase and often it becomes too hard to implement properly. The harsh reality is that software has to be designed with error handing in mind. You have to carefully think about, how errors are generated and passed along from the business logic to the UI. This is not as easy as it might seam sometimes. Key value coding validation allows you to integrate error checking right into your data model using its validate methods. The idea behind validation methods is pretty straight-forward. For every property, a method called <code>validate&lt;PropertyName&gt;:forKey:error:</code> contains code that checks if the new value supplied is valid for that property. It returns YES or NO depending on whether or not the value is acceptable and if not, an error can be returned in the last parameter.  Additionally, the value, provided to the method via the first parameter can be modified during the function's executing. The idea is that, if validation fails, a different value might be supplied that will not fail the checks but is semantically identical to the one provided. For example - let's say a numeric field is being validated, but the user has specified a string (@"15 km") instead of a number. The validation method realizes that it received an object of the wrong type, but instead of returning NO and passing an NSError object, it first tries to convert the string to a number, just in case it contains same valid information. In this case, it would find the number 15 in the string and decide to use that. It might even realize that the user has specified kilometers and it needs a distance in meters and multiply 15 by 1000 to get 15000 as a final result. This can, no doubt, be useful in many applications. Unfortunately, Key value coding validation will not automatically invoke the validate method every time a property setter is called. You have to explicitly validate the value before calling the setter.

<code>NSError* validationError = nil;</code>

[accident validateValue:&amp;valueToValidate forKey:propertyKey error:&amp;validationError];

if (validationError == nil)

{

[accident setValue:valueToValidate forKey:propertyKey];

}

The down side of using validation methods is that you have to write one for every property of every data classes you wish to use. That can be extremely tedious. So tedious that I wrote a bash script that generates them for me. It recognizes common field types such as NSString*, NSDate and NSNumber and creates a template suitable for that type. For all other data types, it just creates the method definition returning YES by default. In order to use it, just supply your .h file (containing the @property definitions) and the script will dump all validate method implementations in the console. If you are interested, here's the code:

<code>#!/bin/bash</code>

function capitalizeFirstLetter

{

word=$1

word="$(tr '[:lower:]' '[:upper:]' &lt;&lt;&lt; ${word:0:1})${word:1}"

}

fileName=$1

allProperties=$(grep @property $fileName)

grep @property $fileName |  while read -r property

do

# handle string properties

isString=$(echo $property | grep NSString)

propertyName=$(echo $property | rev | cut -d' ' -f 1 | rev | cut -d';' -f 1)

propertyName="$(tr '[:lower:]' '[:upper:]' &lt;&lt;&lt; ${propertyName:0:1})${propertyName:1}"

if [[ ! -z $isString ]]

then

echo -e "- (BOOL)validate$propertyName:(NSString**)value\n error:(NSError**)validationError\n{\n\tif ((*value).length == 0)\n\t{\n\t\tNSError* error = [NSError errorWithCode:1\n\t\t domain:(NSString*)??? \n\t\tdescription:NSLocalizedString(@\"???\", @\"\")];\n\t\t*validationError = error;\n\t\treturn NO;\n\t}\n\treturn YES;\n}"

fi

# handle date properties

isDate=$(echo $property | grep NSDate)

if [[ ! -z $isDate ]]

then

echo -e "- (BOOL)validate$propertyName:(NSDate**)date\n error:(NSError**)validationError\n{\n\tif (*date == nil)\n\t{\n\t\tNSError* noDateError = [NSError errorWithCode:1\n\t\t domain:(NSString*)??? \n\t\tdescription:NSLocalizedString(@\"???\", @\"\")];\n\t\t\t*validationError = noDateError;\n\t\t\treturn NO;\n\t\t}\n\t\telse if( [*date compare:[NSDate date]] == NSOrderedDescending )\n\t\t{\n\t\t\tNSError *futureDateError = [NSError errorWithCode:1\n\t\t domain:(NSString*)??? \n\t\tdescription:NSLocalizedString(@\"???\", @\"\")];\n\t\t\t*validationError = futureDateError;\n\t\t\treturn NO;\n\t\t}\n\t\treturn YES;\n}\n"

fi

isArray=$(echo $property | grep "Array")

if [[ ! -z $isArray ]]

then

echo -e "- (BOOL)validate$propertyName:(NSArray**)array\n\t\terror:(NSError**)validationError\n{\n\tif ((*array).count == 0)\n\t{\n\t\t// Must have at least one object\n\t\tNSError* noDateError = [NSError errorWithCode:1\n\t\t domain:(NSString*)??? \n\t\tdescription:NSLocalizedString(@\"???\", @\"\")];\n\t\t*validationError = noDateError;\n\t\treturn NO;\n\t}\n\treturn YES;\n}\n"

fi

if [[ -z $isString &amp;&amp; -z $isArray &amp;&amp; -z $isDate ]]

then

echo -e "- (BOOL)validate$propertyName:(NSObject**)value\n\t\terror:(NSError**)validationError\n{\n\t// TODO: implement validation for $propertyName \n\treturn YES;\n}\n"

fi

echo -e '\n'

done

It's not going to win any prizes, but it works for me.

The good news is that validation methods are the only ones that you have to explicitly define. Setters and getters are handled by the methods generated by @property and @synthesize.

I guess that's it. That's all you need to do to use Key value coding validation. I hope you found this tutorial useful and think about times when you could have used key value observing for your applications. In my next post, I'll be covering Key-Value Observing - another great, more advanced feature of the Cocoa framework. Thanks for reading.

&nbsp;