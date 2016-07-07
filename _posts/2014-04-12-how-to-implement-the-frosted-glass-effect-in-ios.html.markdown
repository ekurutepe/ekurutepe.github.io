---
title: How to Implement the Frosted Glass Effect in iOS
date: 2014-04-12 00:00 UTC
tags: ios, cocoa, development
---

The frosted glass effect is a central design element in iOS 7 and there are various ways how you can add it to views in your app. I have been working a lot with blur effects lately and decided to write up what I've learned in a series of posts. Let's start with the simple case of creating a static frosted glass effect before moving on to blurred backgrounds with animations.

A `UIToolbar` instance can be used as a superview instead of a normal `UIView` to provide a frosted glass effect, however with iOS 7.1 it has become much difficult to configure the desired effect correctly. There are also <a href="https://github.com/JagCesar/iOS-blur/issues/25">reports</a> of Apple rejecting apps going this route.

A better approach is to take advantage of the `drawViewHierarchyInRect:afterScreenUpdates:` method in `UIView` to efficiently capture an image of the background and use the `UIImage+ImageEffects` category by Apple to obtain a customized blur effect to achieve the frosted glass effect. This is best achieved in the `willMoveToSuperview:` method of a `UIView` subclass which implements the frosted glass effect:

```objc
- (void)willMoveToSuperview:(UIView *)newSuperview
{
    [super willMoveToSuperview:newSuperview];
    if (newSuperview == nil) {
        return;
    }
    UIGraphicsBeginImageContextWithOptions(newSuperview.bounds.size, YES, 0.0);
    [newSuperview drawViewHierarchyInRect:newSuperview.bounds afterScreenUpdates:YES];
    UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
    UIImage *croppedImage = [UIImage imageWithCGImage:CGImageCreateWithImageInRect(img.CGImage, self.frame)];
    UIGraphicsEndImageContext();

    self.backgroundImage = [croppedImage applyBlurWithRadius:11
                                                   tintColor:[UIColor colorWithWhite:1 alpha:0.3]
                                       saturationDeltaFactor:1.8
                                                   maskImage:nil];
}
```

Since the background is computed once the frosted glass view moves into a superview, changing the frame of the frosted glass view will destroy the illusion. So make sure that you use this trick only for views where you know the frame or the background will not be changing.

The source code for this simple frosted glass view is on <a href="https://github.com/ekurutepe/iOS-Blur-Examples">Github</a>. In the next post in this series, I'll show how this view can be extended to support changing the frame over a static background.