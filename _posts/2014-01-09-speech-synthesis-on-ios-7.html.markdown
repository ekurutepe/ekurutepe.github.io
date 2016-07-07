---
title: Speech Synthesis on iOS 7
date: 2014-01-09 00:00 UTC
tags: ios, cocoa, development
---

One of the new APIs introduced with iOS 7 is the new <a href="https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVSpeechSynthesizer_Ref/Reference/Reference.html">speech synthesizer</a> in the AVFoundation framework. It went relatively unnoticed among the flood of more significant changes in iOS 7.

I recently received a review for my app <a href="https://itunes.apple.com/us/app/the-seven-minute-workout/id650872326?mt=8&amp;uo=4">The Seven Minute Workout</a> asking for spoken announcement of exercises before they start. What a nice idea, I thought let me record somebody (read: a native speaker, preferably female) saying the names of the exercises, add the audio files to the app bundle and play them when appropriate. Then a Eureka moment came: speech synthesis!

After a bit of Googling and finding out about the new Apple Speech Synthesizer, the documentation for the API was quite straight-forward. There are two classes involved in making your iPhone speak: <a href="https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVSpeechUtterance_Ref/Reference/Reference.html">`AVSpeechUtterance`</a> which represents what is being said and <a href="https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVSpeechSynthesizer_Ref/Reference/Reference.html">`AVSpeechSynthesizer`</a> which does the speaking itself.

Here is some simple example code:

```objc
    AVSpeechUtterance *utterance = [AVSpeechUtterance speechUtteranceWithString:@&quot;Hello World!&quot;];
    utterance.voice = [AVSpeechSynthesisVoice voiceWithLanguage:@&quot;en-US&quot;];
    utterance.rate = 0.70*AVSpeechUtteranceDefaultSpeechRate;

    AVSpeechSynthesizer *synth = [[AVSpeechSynthesizer alloc] init];
    [synth speakUtterance:utterance];
```

The code itself is pretty straight-forward. The only important detail that you should pay attention is to use the correct voice, which defaults to the locale specific language and the speed of the utterance, which is probably optimized for turn-by-turn navigation was way too fast for the purposes of <a href="https://itunes.apple.com/us/app/the-seven-minute-workout/id650872326?mt=8&amp;uo=4">The Seven Minute Workout</a>.