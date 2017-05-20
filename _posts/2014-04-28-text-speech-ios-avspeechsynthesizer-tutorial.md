---
id: 202
title: 'Text to speech in iOS - AVSpeechSynthesizer tutorial'
date: 2014-04-28T11:21:13+00:00
author: n.sobadjiev@gmail.com
layout: post
guid: http://devmonologue.com/ios/?p=202
permalink: /tutorials/text-speech-ios-avspeechsynthesizer-tutorial/
categories:
  - Tutorials
tags:
  - AVSpeechSynthesizer
  - AVSpeechUtterance
  - ios
  - ObjC
  - Objective-C
  - text to speech
---
# Text to speech in iOS

Did you know that there is a speech synthesis API available to iOS developers? I certainly didn't. But there is! Starting with iOS7 the [AVSpeechSynthesizer][Apple docs on AVSpeechSynthesizer] class provides functionality for converting text into audio.

Text to speech in iOS is not something new. In fact, it has always been a part of the iPhone's accessibility options, under the name [VoiceOver][Apple docs on VoiceOver].

![Screenshot displaying the VoiceOver options in the Settings application]({{ site.url }}/images/2014/04/voiceoversettings.png "VoiceOver settings")

Additionally, voice recognition has been on iOS for a while. Just tapping on the microphone icon on the keyboard allows you to dictate instead of typing text. However, up until now we, as developers, had little access to those powerful features.

![Screenshot showing the voice recognition button in UIKeyboard]({{ site.url }}/images/2014/04/voicerecognitioninuikeyboard.png "Voice recognition in UIKeyboard")

Using text to speech with [AVSpeechSynthesizer][Apple docs on AVSpeechSynthesizer], you will hear the familiar voice that Siri uses. But unlike Siri, your speech synthesizing will be available even when your device is not connected to the internet, which can be really useful. Now, I don't know about you, but I feel that speech generated by Siri might sound a little better. It is just a bit unnatural to me. I know that text to speech is an inherently difficult problem to solve, and it will improve over time, but I think Apple could have done a little better.

# [AVSpeechUtterance][Apple docs on AVSpeechUtterance]

The AVSpeechUtterance class takes a central position in Apple's text to speech API. _Btw, utterance means "spoken word, statement, or vocal sound", if you are wondering_.
So, an AVSpeechUtterance instance encapsulates all data needed in order for iOS to speak some text. This includes:

* [`speechString`] The actual text data as `NSString`
* [`rate`] The speed (rate) at which the text is spoken
* [`pitchMultiplier`] The pitch of the voice
* [`voice`] The voice to be used (American, British English, Australian, Japaneese etc.)
* [`volume`] Volume
* Post [`postUtteranceDelay`] and pre [`preUtteranceDelay`] utterance delay

Though most of these properties don't seem that important, in my experience, some of them require some calibration. For instance, the `rate` property that configures the speed of the speech just doesn't work very well with it's default value. The possible values are between `0.0f` and `1.0f` with the default value being `0.5`. In my opinion, this is a bit too fast. It sound artificial and difficult to listen to. So I'd probably prefer a value around `0.15` or so.

Additionally, modifying the `pitchMultiplier` property might give you very different results when it comes to the voice used for the output. Values are between `0.5f` and `2.0f` with `1.0f` as the default. You might want to experiment with those property and figure out what works best for you.

# [AVSpeechSynthesizer][Apple docs on AVSpeechSynthesizer]

AVSpeechSynthesizer is the class that does the actual text to speech conversion. Basically, you feed it AVSpeechUtterance instances and it speaks them for you. There is not much more to it than that. It provides an interface for pausing and stopping playback and it also queues utterances. This means that if you add another text while the current is still playing, it will wait until it's done with the first one and continue with the second.

An interesting feature of AVSpeechSynthesizer is that you can customize how speech is paused. The actual pausing is done using the

```objc
- (BOOL)pauseSpeakingAtBoundary:(AVSpeechBoundary)boundary
```

method. By using the `boundary` parameter, you can tell AVSpeechSynthesizer to either stop speaking immediately, or wait until it's done saying a word to avoid the audio cutting in the middle of it.

# [AVSpeechSynthesizerDelegate][Apple docs on AVSpeechSynthesizerDelegate]

As any other serious Objective-C class, AVSpeechSynthesizer has a delegate. Using that, your classes can get notified whenever the text to speech starts, stops and pauses. The following methods are available:

* `– speechSynthesizer:didCancelSpeechUtterance:`
* `– speechSynthesizer:didContinueSpeechUtterance:`
* `– speechSynthesizer:didFinishSpeechUtterance:`
* `– speechSynthesizer:didPauseSpeechUtterance:`
* `– speechSynthesizer:didStartSpeechUtterance:`
* `– speechSynthesizer:willSpeakRangeOfSpeechString:utterance:`

These are all optional and should be fairly easy to understand. The only thing that, I feel, needs a little more discussion is the last one. It will fire everytime the text to speech algorithm goes to the next unit. And by "unit" I mean "word" mostly. This can be really useful if you want to provide feedback regarding the current position that is being synthesized. For instance, you might want to hightlight that particular portion of the paragraph in your UI or simulate a conventional music player and indicated the current position within the audio file.

# AVSpeechSynthesizer example

I've created a [sample application][SpeechSynthesizerDemo link] for you that showcases some of the aspects of using AVSpeechSynthesizer. It is rather minimalistic and has some issues, but it should be enough for you to see this class in action. It contains a text view where you can type the text you want AVSpeechSynthesizer to speak and a couple of slider you can use to change the pitch and rate. It should look something like this:

![Screenshot of the SpeechSynthesizerDemo application]({{ site.url }}/images/2014/04/speechsynthesizerdemo.png "SpeechSynthesizerDemo in action")

# AVSpeechSynthesizer alternatives

There are many reasons that might make you consider an alternative to AVSpeechSynthesizer. For one thing, there are services that offer a greater quality when it comes to voices and synthesizing human speech. Or maybe you need to save the audio in the file system. The point is that the text to speech in iOS is not always the best solution given the circumstances. In my research, I found several services that are worth mentioning.

_This list is, by no means, complete. These are just services I found in my search for text to speech in iOS. If you have something to add, please let me know in the comment section._

1. [iSpeech][iSpeech website]

 iSpeech is a commercial text to speech service that, in my opinion, offers the best audio generation quality. The voices sound very natural. Well, there is definitely a difference between a real human voice and iSpeech's synthesized speech, but that's a good as it gets with the technology we have nowadays.

 iSpeech has an iOS framework (among other platforms) that provides an API similar to AVSpeechSynthesizer. It's drawback is that it does not allow you to save an audio file containing the generated speech, which might be a problem considering that iSpeech requires an internet connection. Also, it is a paid service, and while I think that they are very flexible when it comes to pricing, it still means that your application will probably not be free.

2. [Voice RSS][voiceRSS website]

 Voice RSS is also an internet text to speech service. It requires you to have a network connection in order to convert text, but at least it allows you to save the audio in a file. It combines words seamlessly and has overall high synthesizing quality, but it provides you with audio formats with low bitrate. This means that whereas the speech sounds relatively natural, the quality of the audio itself suffers.

3. [OpenEars][openEars website]

 OpenEars is an opensource library for speech recognition and synthesis on iOS. It's major advantage is that it does text to speech locally and does not rely on the network as any point. The bad news is that since it has to store everything locally, it is around 50 MB in size. This can be too much for a small iOS application. Also, you might want to run the sample project beforehand in order to determine if the level of quality and accuracy is enough for your needs, because it is certainly not as advanced as the services mentioned above. Despite that, if you need an offline text to speech solution and you need to support iOS versions below 7, OpenEars can be the way to go for you.

I hope you found this article useful and please share in the comment section below how you plan to use text to speech in your iOS applications. It is an exciting topic and I **know** you can come up with some great ideas for it.

Thanks for reading!

[Apple docs on AVSpeechSynthesizer]: https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVSpeechSynthesizer_Ref/Reference/Reference.html#//apple_ref/occ/cl/AVSpeechSynthesizer
[Apple docs on VoiceOver]: https://www.apple.com/accessibility/ios/voiceover/
[Apple docs on AVSpeechUtterance]: https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVSpeechUtterance_Ref/Reference/Reference.html#//apple_ref/occ/cl/AVSpeechUtterance
[Apple docs on AVSpeechSynthesizerDelegate]: https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVSpeechSynthesizerDelegate_Ref/Reference/Reference.html#//apple_ref/occ/intfm/AVSpeechSynthesizerDelegate/speechSynthesizer:willSpeakRangeOfSpeechString:utterance:
[iSpeech website]: http://www.ispeech.org
[voiceRSS website]: http://www.voicerss.org
[openEars website]: http://www.politepix.com/openears/
[SpeechSynthesizerDemo link]: {{ site.url }}/images/2014/04/SpeechSynthesizerDemo.zip