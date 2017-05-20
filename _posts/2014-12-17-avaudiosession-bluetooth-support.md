---
id: 328
title: AVAudioSession bluetooth support
date: 2014-12-17T12:08:00+00:00
author: Nikola Sobadjiev
layout: post
guid: http://devmonologue.com/ios/?p=328
permalink: /tutorials/avaudiosession-bluetooth-support/
categories:
  - Tutorials
tags:
  - AVAudioSession
  - bluetooth
  - ios
  - Objective-C
---
<a href="{{ site.url }}/images/2014/12/bluetooth_logo.jpg"><img class="aligncenter size-medium wp-image-329" src="{{ site.url }}/images/2014/12/bluetooth_logo-300x207.jpg" alt="Bluetooth's logo" width="300" height="207" /></a>
<h2>Adding Bluetooth audio support to an iOS app</h2>
If you've come here via an internet search, I assume you're already pulling your hair out trying to add bluetooth support to your app. I can almost feel your pain. I've been through it all. I will give you a few pointers about implementing  AVAudioSession bluetooth support, but keep in mind that I'm no expert. I don't, by any means, feel like I know much about iOS audio. I just had the fortune to successfully get AVAudioSession bluetooth working and decided to share my knowledge with the hope that some day I might be able to help someone though tough times. So bear with me :)
<h2>AVAudioSession bluetooth support source code</h2>
Adding AVAudioSession bluetooth support actually takes a lot less code than you might think. In fact, the whole audio sessions API is so simplistic that everything happens either very easy or incredibly hard. Guess which category AVAudioSession bluetooth support falls in?

Adding bluetooth to an AVAudioSession involves several features as far as I'm concerned. Lets go through them all:
<h2>Enabling bluetooth audio</h2>
First of all, you need to specifically instruct AVAudioSession that you allow sound to be routed to bluetooth devices. This can be done while setting your audio category:
<pre class="brush:objc">[session setCategory:AVAudioSessionCategoryPlayAndRecord 
         withOptions:AVAudioSessionCategoryOptionAllowBluetooth
               error:&amp;error];</pre>
Remember, your audio session category tells iOS how you plan on using sound within your app. For instance, <code>AVAudioSessionCategoryPlayAndRecord</code> means that you want to both play and record audio. Anyway, what we are interested in, is the options parameter. By adding <code>AVAudioSessionCategoryOptionAllowBluetooth</code>, we allow iOS to play our app's audio on a bluetooth audio device.

<strong>Note:</strong> I've seen projects where a category is set on an AVAudioSession in multiple places. If I've learned anything about audio in iOS is that you <strong>don't use setCategory: lightly!</strong> In fact, in most cases you should only set is once. Why? For one thing it is a complex operation. If it's executed on the main thread, it will make your UI unresponsive for a second. But more importantly, setting the category all over the place might lead into strange behavior - you will never know when audio will be going to the speaker or the earphone...
<h2>Switching between audio routes</h2>
When you are implementing AVAudioSession bluetooth support, you will probably want to be able to switch between audio devices. And there doesn't seem to be a clearly defined way to do that in the documentation.

There are several ways to do it. However, what I found to be the best working one is setting the <span style="text-decoration: underline;">preferred audio input</span>. Here are some examples:
<h3>Switching to bluetooth</h3>
<pre class="brush:objc">- (AVAudioSessionPortDescription*)bluetoothAudioDevice
{
    NSArray* bluetoothRoutes = @[AVAudioSessionPortBluetoothA2DP, AVAudioSessionPortBluetoothLE, AVAudioSessionPortBluetoothHFP];
    return [self audioDeviceFromTypes:bluetoothRoutes];
}

- (AVAudioSessionPortDescription*)builtinAudioDevice
{
    NSArray* builtinRoutes = @[AVAudioSessionPortBuiltInMic];
    return [self audioDeviceFromTypes:builtinRoutes];
}

- (AVAudioSessionPortDescription*)speakerAudioDevice
{
    NSArray* builtinRoutes = @[AVAudioSessionPortBuiltInSpeaker];
    return [self audioDeviceFromTypes:builtinRoutes];
}

- (AVAudioSessionPortDescription*)audioDeviceFromTypes:(NSArray*)types
{
    NSArray* routes = [[AVAudioSession sharedInstance] availableInputs];
    for (AVAudioSessionPortDescription* route in routes)
    {
        if ([types containsObject:route.portType])
        {
            return route;
        }
    }
    return nil;
}

- (BOOL)switchBluetooth:(BOOL)onOrOff
{
    NSError* audioError = nil;
    BOOL changeResult = NO;
    if (onOrOff == YES)
    {
        AVAudioSessionPortDescription* _bluetoothPort = [self bluetoothAudioDevice];
        changeResult = [[AVAudioSession sharedInstance] setPreferredInput:_bluetoothPort
                                                     error:&amp;audioError];
    }
    else
    {
        AVAudioSessionPortDescription* builtinPort = [self builtinAudioDevice];
        changeResult = [[AVAudioSession sharedInstance] setPreferredInput:builtinPort
                                                     error:&amp;audioError];
    }
    return changeResult;
}</pre>
Wow, that was a lot of code! Well, the important bit is setting the preferred input in the last method. The other code is just to get the right AVAudioSessionPortDescription object for speaker, earphone or bluetooth.
<h3>Switching to speaker</h3>
While there are other way to do that, this is what I prefer to use:
<pre class="brush:objc">- (BOOL)switchSpeaker:(BOOL)onOrOff
{
    NSError* audioError = nil;
    BOOL changeResult = NO;
    if (onOrOff == YES)
    {
        changeResult = [[AVAudioSession sharedInstance] overrideOutputAudioPort:AVAudioSessionPortOverrideSpeaker
                                                           error:&amp;audioError];
    }
    else
    {
        AVAudioSessionPortDescription* builtinPort = [self builtinAudioDevice];
        changeResult = [[AVAudioSession sharedInstance] setPreferredInput:builtinPort
                                                                    error:&amp;audioError];
    }
    return changeResult;
}</pre>
<h3>Switching to earpiece</h3>
This one is easy. It's just the opposite of switching to speaker:
<pre class="brush:objc">- (BOOL)switchEarphone:(BOOL)onOrOff
{
    return [self switchSpeaker:!onOrOff];
}

AVAudioSession bluetooth support can be a daunting task. And there isn't much information about it in the internet. I hope this article helps you with your task. Thanks for reading!</pre>