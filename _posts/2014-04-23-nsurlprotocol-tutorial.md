---
id: 192
title: NSURLProtocol tutorial
date: 2014-04-23T11:29:50+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=192
permalink: /tutorials/nsurlprotocol-tutorial/
categories:
  - Tutorials
tags:
  - ios
  - NSURLProtocol
  - ObjC
  - Objective-C
---
## What is a [NSURLProtocol][Apple docs on NSURLProtocol]?

According to Apple's documentation:

> NSURLProtocol is an abstract class that provides the basic structure for performing protocol-specific loading of URL data. Concrete subclasses handle the specifics associated with one or more protocols or URL schemes.

More intuitively, an NSURLProtocol is something that can be used in order to modify the way an iOS application handles URL requests. Normally, the operating system supports schemas such as FTP and HTTP out of the box, but if you need to add another one that cannot be handled natively, you can create an NSURLProtocol subclass to do it for you. But why should you go through all that? What's so special about this superclass? Well, since it is part of the URL loading mechanism in iOS, you have the power to change the way requests are handled at a global level. This will allow you to register your NSURLProtocol subclass with the system and have it work for **all** connections initiated by your application, even if those connections are created and started beyond your control. This can be a life saver in some situations.

### A real world NSURLProtocol example

It this article, we are going to focus mainly on an example from a problem that I personally encountered recently. I was working on an [EPUB][EPUB specs] reader application. If you don't know what an EPUB is, it is a popular file format for digital publications (similar to PDF). It is used in [iBooks][iBooks] and other platforms (not including the Kindle) for book reading. There is a whole set of specifications describing this format in detail, but put short, an EPUB is a zip archive, containing a bunch of XHTML files representing all pages of the book, in addition to some special files such as the table of contents and index of all included items. It is faily straight-forward. You just need to open the archive, parse several XML files and display the actual book in a UIWebView. The source code is available at [my github page][SimpleEpubReader github page].

Sounds easy enough, right? Well, things got a little more complex down the road. First of all, I never found the need to extract the whole archive before starting work on it, as the [zipzap framework][zipzap's github page], which I used for handling the zip files, supports listing and reading data without unzippping everything. However, as soon as I was able to populate a web view with the data from the EPUB file, I noticed a major flaw in my decision. I was reading an HTML file from the archive and displaying it on screen, but failed to realize that web pages nowadays are rarely just one file. There are also CSS files, images and other resources that make the them look good on screen. And since all of them were hidden inside the zip file, the WebKit was unable to find and use them. Faced with the alternative of reading HTML files in order to know which resources to extract and save on the file system, I quicky realized that NSURLProtocol is the way to go. It would allow me to use a custom URL schema (I opted for "epub://") and register an NSURLProtocol subclass to modify the way a UIWebView loads it's resources. More specifically, as soon as it encountered a resource using my schema, it would try to find it inside the EPUB archive, rather than the file system or the network.

To be honest, I'm not the first one to write about NSURLProtocol. In fact, I learned about it reading some excelent blog posts on the subject, such as [NSURLProtocol Tutorial][raywenderlich NSURLProtocol tutorial] from Ray Wenderlich's website. However, in my honest opinion, every NSURLProtocol problem is different and every bit of information on the subject is useful. Also, most resources available on the internet seem to focus around changing some parameters of the NSURLRequest before sending it. In Ray's article, for instance, a sample project is built that implements web browser application and caches all loaded pages so that they can be viewed online. But my guess is that many of you might have to do more than intercept network requests, but rather have a completely different loading mechanism. So here we go!

### Implementing NSURLProtocol methods

There are a few methods that need to be implemented in order to conform to NSURLProtocol:

* `+ (BOOL)canInitWithRequest:(NSURLRequest *)request `

This class method is called by the system for all registered URL protocols to determine which of them can handle a particular request. Given an NSURLRequest, classes are expected to return a BOOL value indicating whether or not they can handle that connection. In my case, I had to check if the URL for that request had "epub" as schema.

* `+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request`

This one is a little strange. According to the documentation:

> It is up to each concrete protocol implementation to define what “canonical” means. A protocol should guarantee that the same input request always yields the same canonical form.
Special consideration should be given when implementing this method, because the canonical form of a request is used to lookup objects in the URL cache, a process which performs equality checks between NSURLRequest objects.

Most often you wouldn't have to worry about it though. It's generally fine to return the same request as provided in the parameters.

* `+ (BOOL)requestIsCacheEquivalent:(NSURLRequest *)a toRequest:(NSURLRequest *)b`

This method is used in order to determine whether of not two request are equal when it comes to caching. This means that if it specifies that they are equivalent, the system will try to get the precached response from the first one. Most of the time, the default implementation is sufficient, so you can just call the same method from the superclass (`[super requestIsCacheEquivalent:a toRequest:b]`).

* `- (void)startLoading`

The startLoading method is the one that actually executes the request. This is where the magic happens and where most of your new functionality will go. In my example, this is where I started retrieving data from the zip archive.

* `- (void)stopLoading`

If you've ever worked with NSURLConnection, you should be quite familiar with the concepts behind the startLoading and stopLoading methods. As you've probably guess already, the latter is used in order to cancel a request. It is, ofcourse, required that you provide an implementation in your NSURLProtocol subclass and it is your responsibility to make sure all tasks are stopped and no further delegate methods called.

There are other methods in NSURLProtocol, but these are enough for most developers. Ofcourse, our work is far from over. We still have to handle our request and notify our "client" that we have finished loading.

### [NSURLProtocolClient][Apple docs on NSURLProtocolClient]

You can think of NSURLProtocolClient as NSURLProtocol's delegate. It is implemented by the object that you will be notifying about changes in your request's loading state. It is accessible via NSURLProtocol's `client` property. Taking a look at it's [reference page][Apple docs on NSURLProtocolClient], you might find that it's methods are rather familiar. In fact, they are almost identical to those in [NSURLConnectionDelegate][[NSURLConnectionDelegate]. The most important ones are:

* `- (void)URLProtocol:(NSURLProtocol *)protocol didReceiveResponse:(NSURLResponse *)response cacheStoragePolicy:(NSURLCacheStoragePolicy)policy;`
* `- (void)URLProtocol:(NSURLProtocol *)protocol didLoadData:(NSData *)data;`
* `- (void)URLProtocolDidFinishLoading:(NSURLProtocol *)protocol;`
* `- (void)URLProtocol:(NSURLProtocol *)protocol didFailWithError:(NSError *)error;`

This is one of the places where implementing a NSURLProtocol subclass about loading local data becomes a bit harder. As you can see, NSURLProtocolClient is considerably more complex than it needs to be in order to supply file system data. Retrieving data there is a lot simpler than returing responses, loading data in chunks and so on. However, the protocol requires it, so we have to create a [stub][definition of stub] that simulates that extra funtionality. For the [Simple EPUB reader project][SimpleEpubReader github page], I used the following code:

```objc
- (void)startLoading
{
    NSString* filePath = [self epubFilePath];
    NSString* zipPath = [self zipPath];
    [epubManager openEpubWithPath:filePath];
    NSError* zipError = nil;
    NSData* fileData = [epubManager dataForFileAtPath:zipPath
                                                error:&zipError];
    [self sendResponseWithData:fileData
                         error:&zipError];
}

- (void)sendResponseWithData:(NSData*)data
                       error:(NSError**)error
{
    if ([self handleError:*error] == NO)
    {
        [self handleResponse:data];
        [self handleResponseData:data];
        [self handleRequestFinished];
    }
}

- (BOOL)handleError:(NSError*)error
{
    if (error != nil)
    {
        [self.client URLProtocol:self
                didFailWithError:error];
        return YES;
    }
    else
    {
        return NO;
    }
}

- (void)handleResponse:(NSData*)data
{
    NSString* mediaType = [epubManager mimeTypeForPath:[self zipPath]];
    NSURLResponse* response = [[NSURLResponse alloc] initWithURL:requestUrl
                                                        MIMEType:mediaType
                                           expectedContentLength:data.length
                                                textEncodingName:@"utf-8"];
    [self.client URLProtocol:self
          didReceiveResponse:response
          cacheStoragePolicy:NSURLCacheStorageAllowed];
}

- (void)handleResponseData:(NSData*)data
{
    [self.client URLProtocol:self
                 didLoadData:data];
}

- (void)handleRequestFinished
{
    [self.client URLProtocolDidFinishLoading:self];
}
```

There are a few points that are worth mentioning in this code:

* We assume that the request is successful every time the EPUB manager doesn't return an error object when trying to read a resource. It there is one, we call the client's `URLProtocol:didFailWithError:` method and pass that same error object as parameter.
* We need to create a fake response object to pass to `URLProtocol:didReceiveResponse:cacheStoragePolicy:`. There are several pieces of information that have to be included:
* The request URL

This should be fairly easy. You just have to save the URL that came with the original request object

* The Mime type

This is a bit complicated. You have to assume that whoever is going to use this response, will want to get the mime type of the response. Many times it will not be easy to what it is. In the EPUB reader, I was able to get the value from a special XML file within the publication (the root file). However, this might not be the case with your project, so you will have to figure out on your own how to get the mime type.

* The data length
* The text encoding

For textual data, you have to provide an encoding type. The Simple EPUB reader project makes the (false) assumption that it is always UTF-8. Again, you will have to work out what works for you and how you can get that information.

### Registering a NSURLProtocol subclass

In order to start handling requests with your newly created class, you need to register it with URL loading system. This is really simple to do with the following statement:

```objc
[NSURLProtocol registerClass:[MyURLProtocolSubclass class]];
```

Put this in a central place in your application so that it is called as soon as it is started. Your app delegate can be a convenient place to do that, unless you find a more sensible spot. Keep in mind that if you register several classes to handle the same request, the iOS will pass it to the first one that returns `YES` from it's `canInitWithRequest:` method and the sequence in which that happens is not quaranteed.

### Conclusion

This pretty much sums up the basics of using NSURLProtocol. If you need any additional assistance, don't hesitate to write me a comment in the comment section below and let me know how you use NSURLProtocol in your applications.

Thanks for reading!

[Apple docs on NSURLProtocol]: https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSURLProtocol_Class/Reference/Reference.html
[zipzap's github page]: https://github.com/pixelglow/zipzap/
[raywenderlich NSURLProtocol tutorial]: http://www.raywenderlich.com/59982/nsurlprotocol-tutorial
[EPUB specs]: http://idpf.org/epub
[Apple docs on NSURLProtocolClient]: https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Protocols/NSURLProtocolClient_Protocol/Reference/Reference.html#//apple_ref/occ/intf/NSURLProtocolClient
[NSURLConnectionDelegate]: https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLConnectionDelegate_Protocol/Reference/Reference.html#//apple_ref/occ/intf/NSURLConnectionDelegate
[definition of stub]: https://en.wikipedia.org/wiki/Method_stub
[SimpleEpubReader github page]: https://github.com/nsobadzhiev/SimpleEpubReader
