---
title: How to make your UITabBarControllers and UINavigationControllers respect your auto-rotation choices
date: 2013-02-07 00:00 UTC
tags: ios, cocoa, development
---

iOS 6 introduced many changes related to view auto-rotation, which basically aim to decouple view controller orientation from the device orientation. While this might sound counter-intuitive, it actually makes a lot of sense and allows for a finer grained control of view controller orientations: A `UITableViewController` in portrait orientation in a pop over contained by a view controller in landscape orientation on an iPad, for instance.

However the `UINavigationController` and `UITabBarController` do not seem to be implementing the new `shouldAutorotate` and `supportedInterfaceOrientations` APIs as expected: regardless of the values returned by your view controllers, they always allow any orientation!

The fix, fortunately, is quite simple and can be added to the standard `UINavigationController` and `UITabBarController` with a simple category, which forwards the values set by the `topViewController` and `selectedViewController` respectively.

Here are the two categories you can add at the beginning of your app delegate:

`UITabBarController`

```objc
@implementation UITabBarController (AutoRotationForwarding)

-(BOOL)shouldAutorotate
{
    if ([self.selectedViewController respondsToSelector:@selector(shouldAutorotate)]) {
        return [self.selectedViewController shouldAutorotate];
    }
    else {
        return YES;
    }
}

- (NSUInteger)supportedInterfaceOrientations {
    if ([self.selectedViewController respondsToSelector:@selector(supportedInterfaceOrientations)]) {
        return [self.selectedViewController supportedInterfaceOrientations];
    }
    else {
        return UIInterfaceOrientationMaskAll;
    }
}

@end
```

and `UINavigationController`

```objc
@implementation UINavigationController (AutoRotationForwarding)

-(BOOL)shouldAutorotate
{
    if ([self.topViewController respondsToSelector:@selector(shouldAutorotate)]) {
        return [self.topViewController shouldAutorotate];
    }
    else {
        return YES;
    }
}

-(NSUInteger) supportedInterfaceOrientations {
    if([self.topViewController respondsToSelector:@selector(supportedInterfaceOrientations)])
    {
        return [self.topViewController supportedInterfaceOrientations];
    }
    return UIInterfaceOrientationMaskPortrait;
}

@end
```

Hope this helps somebody
