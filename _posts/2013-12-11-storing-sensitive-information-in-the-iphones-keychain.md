---
id: 32
title: Using the iOS Keychain for storing sensitive information
date: 2013-12-11T21:58:28+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=32
permalink: /ios/storing-sensitive-information-in-the-iphones-keychain/
categories:
  - iOS
tags:
  - ios
  - keychain
  - security
  - storage
---
Many applications in the AppStore handle account information and other private data. This means that they take responsibility to handle it properly and keep it in a secure location. So how do you make sure no one will be able to steal the user's credentials. This question is simple, but it's not easy.
<h3>Data protection in iOS:</h3>
Normally, most information on iOS devices is encrypted and unavailable to anyone apart from the application that owns it. Also every app runs in it's own "sandbox". This means that every application runs as if it is the only one on the device. It was it's own file system, it's own resources etc. For the most part it's completely isolated from everything on the device that it did not create. That's all very nice (for security), but your data is certainly not safe that way because of jailbreaking. Jailbreaking is the process of gaining root access to your device which gives you the ability to access anything you want. You can view and modify files, configs and you can even tamper with your application's binaries. So if you save anything in your sandbox file system, it will be unprotected from jailbreaking.
<h3>NSUserDefaults:</h3>
Using NSUserDefault to store credentials is a little more secure, but information stored there can still be accessed. It is OK for  keeping some persistent information, but it's not the place you want to use for holding passwords or account information.
<h3>iOS Keychain</h3>
Similar to the KeychainAccess application on your Mac, the iOS Keychain can store sensitive information on your behalf. I'm sure that given enough time, the right people will be able to break into it too, but generally this is the place you would want to keep your passwords. The Keychain API provides functionality for storing different types of account information (generic passwords, internet passwords, certificates, keys and identities to be exact) and all of them support many types of fields allowing all aspects of those particular credentials to be fitted within a single record.
<h3>Using the Keychain services API</h3>
Unfortunately, using the API for storing data in the keychain is not a easy as many of you would hope. It is definitely not simple like NSUserDefaults. First of all...it's written in C (please, don't close the tab, it's only going to hurt for a while). Secondly, there are tons of options and constants to configure which can be really frustrating. If you decide to use this API, I would recommend that you find a tutorial that has provided some good sample code. You can find just that <a href="http://www.raywenderlich.com/6475">here</a>. If you import this source to your project, it will not be difficult to have a working password storage in no time. However, in my experience, getting all the options and storing the right information in the right place for your particular task takes some fiddling. But if we will not be talking about the iOS Keychain services API, what will we be talking about? I'm glad you asked!
<h3>Generic Keychain</h3>
Since Apple knew that many iOS/Mac developers run from C as if it was the plague, they were kind enough to provide a wrapper for it in the form of a sample project called Generic Keychain (available <a href="https://developer.apple.com/library/ios/samplecode/GenericKeychain/Introduction/Intro.html#//apple_ref/doc/uid/DTS40007797">here</a>). It almost exclusively handles the "generic password" type mentioned above, but that's good enough for most people (for a list describing what kind of fields each type can have, please refer to <a href="https://developer.apple.com/library/ios/documentation/Security/Reference/keychainservices/Reference/reference.html#//apple_ref/doc/constant_group/Item_Class_Value_Constants">this page</a> within the iOS reference).

The Generic Keychain sample code contains the KeychainItemWrapper class. This is where all the heavy-lifting is done. The class handles almost all interactions with the keychain services leaving you with a simple, easy to use, NSDictionary style interface. For instance, if I wanted to store my account information, I would write something like this:
<pre class="brush:objc">KeychainItemWrapper* keychain = [[KeychainItemWrapper alloc] initWithIdentifier:@"nick@gmail.com" accessGroup:nil];
[keychain setObject:@"myUserName" forKey:kSecAttrAccount];
[keychain setObject:@"I love chicken" forKey:kSecValueData];
[keychain setObject:@"email" forKey:kSecAttrService];</pre>
Let me explain. The first line creates a keychain wrapper instance that we will be using to store information. You don't have to create a new instance every time, in fact, in Apple's sample they reuse a single object.

The identifier is a string that identifies the record in the iOS keychain. It doesn't have to be unique within your application, but it will be something you will be using to later find that particular record.

The access group can be used to allow access to items in the keychain to other applications (more information on sharing access <a href="https://developer.apple.com/library/ios/documentation/Security/Reference/keychainservices/Reference/reference.html#//apple_ref/c/func/SecItemAdd">here</a>). Passing nil will make the entry only available within the app that created it.

Using the setObject:forKey method you will be able to add/modify properties within the record. In the example above, a username (kSecAttrAccount), a password (kSecValueData) and a service name (kSecAttrService) is set, but you can add any of the properties described <a href="https://developer.apple.com/library/ios/documentation/Security/Reference/keychainservices/Reference/reference.html#//apple_ref/doc/uid/TP30000898-CH4g-SW7">here</a>. Please note that not every password type supports all keys (a generic password does not support kSecAttrProtocol for instance). After each added property the keychain item is saved so you don't have to worry about doing that yourself. That's it!

Now, how about accessing keychain data? Normally, using the iOS Keychain services API, you'd create a dictionary containing all the known properties and query the keychain for entries matching that information. The same is (kind of) true for KeychainItemWrapper:
<pre class="brush:objc">KeychainItemWrapper* keychain = [[KeychainItemWrapper alloc] initWithIdentifier:@"nick@gmail.com" accessGroup:nil];
[keychain setObject:@"email" forKey:kSecAttrService];
NSString* username = [keychain objectForKey:kSecAttrAccount];
NSString* password = [keychain objectForKey:kSecValueData];</pre>
Here, the key is that we provided the keychain wrapper with the information we had on the item we wanted to find (the identifier and the service) and then, started requesting information from the found object using the objectForKey: method.

I guess that's all. As you can see, using Apple's Generic Keychain sample is really simple and if you don't require a lot of control over your password storage, I would recommend it over the iOS Keychain services API. I learned this the hard way, struggling with it a few hours and then switching to KeychainItemWrapper.

Once again, thanks for reading.