---
id: 224
title: Unit testing in Swift
date: 2014-06-10T20:56:08+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=224
permalink: /test-driven-development/unit-testing-swift/
categories:
  - Swift
  - Test Driven Development
tags:
  - ios
  - OCUnit
  - Swift
  - TDD
  - Test Driven Development
---
If you haven't realized it already, I'm a fan of test driven developement. I'm still learning, but I'm trying to introduce it into my coding practice more and more. And for that reason, one of the first things I thought about when I learned about Swift was 

`But wait, how are we going to do unit testing in Swift???`

I wasn't sure if they planned on porting OCUnit from Objective-C. And since you are reading this, I assume you are interested in finding out too. So there you go... they did!

I'm going to keep this post short as there isn't much to talk about. Apple did a really good job making sure we don't have to learn another framework - unit testing in Swift is as close to it's Objective-C equivilent as humanly possible. If you are not familiar with OCUnit, check out the [series of posts][TDD category in devmonologue] I wrote about it.

# Unit testing in Swift via OCUnit

If you've already used OCUnit, very little of your workflow will change. Let's see what you need to do.

If you are setting up a new project, you will, ofcourse, have a testing target already configured by Xcode. On the other hand, if you are working on an existing one, you will be able to use the target that you already have.

Let's try to create an new test case class. As always, open the "New file" wizard in Xcode and choose "Test Case Class".

![New File Template]({{ site.url }}/images/2014/06/new_file_template.png "The new file wizard in Xcode, with Test Case Class selected")

After that, we are presented with some unit test specific options. Choose a name for your new file, leave the superclass as XCTTestCase and in the language field, select "Swift"

![New Swift Test Case File]({{ site.url }}/images/2014/06/new_swift_test_case_file.png "Screenshot of the new file wizard in Xcode showing test case class specific options - superclass, name and language")

Once you do that, you will be presented with this beautiful template:

	import XCTest

	class SwiftUnitTest: XCTestCase {

    	override func setUp() {
        	super.setUp()
        	// Put setup code here. This method is called before the invocation of each test method in the class.
    	}
    
    	override func tearDown() {
        	// Put teardown code here. This method is called after the invocation of each test method in the class.
        	super.tearDown()
    	}

    	func testExample() {
        	// This is an example of a functional test case.
        	XCTAssert(true, "Pass")
    	}

    	func testPerformanceExample() {
        	// This is an example of a performance test case.
        	self.measureBlock() {
            // Put the code you want to measure the time of here.
        	}
    	}

	}

Looks familiar? Well, it should. It has everything an Objective-C unit test had - an XCTestCase superclass, a `setUp` and `tearDown` method and actual testing methods. Additionally, all assertion functions that you know have been ported to Swift.

* XCTAssert
* XCTAssertEqual
* XCTAssertEqualObjects
* XCTAssertEqualWithAccuracy
* XCTAssertFalse
* XCTAssertGreaterThan
* XCTAssertGreaterThanOrEqual
* XCTAssertLessThan
* XCTAssertLessThanOrEqual
* XCTAssertNil
* XCTAssertNotEqual
* XCTAssertNotEqualObjects
* XCTAssertNotEqualWithAccuracy
* XCTAssertNotNil
* XCTAssertTrue

So, don't be intimidated when it comes to unit testing in Swift. Setting it up is really easy if you have previous experience with OCUnit. Most of the frustration I had was related to the language itself - every new begining is hard. But once you come into terms with Swift's synthax, you should be alright.

Finally, let's finish this article with a demonstration. You will be able to get a glimpse of the very first unit test I ever wrote in Swift. It's designed to test an applications's AppDelegate. I like to keep functionality in it to a minimum (as should you). In fact, the only thing it does is handle opening URLs from other applications and copying the referenced file in the documents directory within the sandbox.

First, let's look at the old Objective-C code:

	@interface DMAppDelegateTests : XCTestCase
	{
		DMAppDelegate* appDelegate;
	}

	@end

	@implementation DMAppDelegateTests

	- (void)setUp
	{
		[super setUp];
    	appDelegate = [[DMAppDelegate alloc] init];
    	[appDelegate application:[UIApplication sharedApplication]
		didFinishLaunchingWithOptions:nil];
	}

	- (void)tearDown
	{
    	appDelegate = nil;
    	[super tearDown];
	}

	- (void)testAbilityToOpenFilesFromOtherApps
	{
    	DMTestableFileSystemManager* fileSystemManager = [DMTestableFileSystemManager new];
    	appDelegate.fileManager = fileSystemManager;
    	XCTAssertNoThrow([appDelegate application:[UIApplication sharedApplication]
    									  openURL:[NSURL URLWithString:@"file://path/to/resource"] 
                            	sourceApplication:@"test.app"
                                	   annotation:nil], @"Should respond to the openURL method");
	}

	- (void)testAbilityToCopyOpenedFilesInTheSandbox
	{
    	DMTestableFileSystemManager* fileSystemManager = [DMTestableFileSystemManager new];
    	appDelegate.fileManager = fileSystemManager;
    	NSURL* openedURL = [NSURL URLWithString:@"file://path/to/resource"];
    	[appDelegate application:[UIApplication sharedApplication]
        				 openURL:openedURL
           	  sourceApplication:@"test.app"
              		 annotation:nil];
    	XCTAssertEqual(fileSystemManager.copiedURL, openedURL, @"The app delegate should ask the file manager to copy the file in the documents dir");
	}

	- (void)testAbilityToCopyOpenedFilesInTheRightDestination
	{
    	DMTestableFileSystemManager* fileSystemManager = [DMTestableFileSystemManager new];
    	appDelegate.fileManager = fileSystemManager;
    	NSURL* openedURL = [NSURL URLWithString:@"file://path/to/resource.epub"];
    	[appDelegate application:[UIApplication sharedApplication]
        				 openURL:openedURL
        	   sourceApplication:@"test.app"
               	   	  annotation:nil];
    	NSArray* paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
		NSString* documentsDirectory = [paths objectAtIndex:0];
    	documentsDirectory = [documentsDirectory stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    	NSString* destinationPath = [documentsDirectory stringByAppendingPathComponent:@"resource.epub"];
    	XCTAssertEqualObjects(fileSystemManager.destinationURL.absoluteString, destinationPath, @"The app delegate should ask the file manager to copy the file in the documents dir");
	}

	@end

Several hundred pages worth of Swift's book and some fiddling around with the synthax and you come up with it's Swift equivilent:

	import XCTest
	import UIKit

	class DMAppDelegateTests: XCTestCase {
    
    var appDelegate: AppDelegate = AppDelegate();

    override func setUp() {
        super.setUp()
        appDelegate.application(UIApplication.sharedApplication(), didFinishLaunchingWithOptions: nil)
    }
    
    override func tearDown() {
        super.tearDown()
    }
    
    func testAbilityToCopyOpenedFilesInTheSandbox() {
        var fileSystemManager = DMTestableFileSystemManager()
        appDelegate.fileManager = fileSystemManager as DMFileSystemManager;
        let openedURL = NSURL(string: "file://path/to/resource");
        appDelegate.application(UIApplication.sharedApplication(), openURL: openedURL, sourceApplication: "test.app", annotation: nil)
        XCTAssertEqual(fileSystemManager.copiedURL!, openedURL, "The app delegate should ask the file manager to copy the file in the documents dir");
    }
    
    func testAbilityToCopyOpenedFilesInTheRightDestination() {
        var fileSystemManager = DMTestableFileSystemManager()
        appDelegate.fileManager = fileSystemManager as DMFileSystemManager;
        let openedURL = NSURL(string: "file://path/to/resource.epub");
        appDelegate.application(UIApplication.sharedApplication(), openURL: openedURL, sourceApplication: "test.app", annotation: nil)
        let documentsDirectoryRaw = NSHomeDirectory() + "Documents";
        let documentsDirectory = documentsDirectoryRaw.stringByAddingPercentEscapesUsingEncoding(NSUTF8StringEncoding)
        let destinationPath = documentsDirectory.stringByAppendingPathComponent("resource.epub")
        XCTAssertEqualObjects(fileSystemManager.destinationURL!.absoluteString, destinationPath, "The app delegate should ask the file manager to copy the file in the documents dir");
    }

	}
	
And that's how you get unit testing in Swift. As always, thanks for reading!
 
[TDD category in devmonologue]: http://devmonologue.com/ios/category/test-driven-development/