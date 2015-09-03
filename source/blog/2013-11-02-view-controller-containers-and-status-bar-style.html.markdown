---
title: View Controller Containers and Status Bar Style
date: 2013-11-02 00:00 UTC
tags: ios, cocoa, development
---

One of the changes in iOS 7 is the new method of determining the status bar style through the `preferredStatusBarStyle` method.

The basic idea is quite simple: the system asks the current view controller for the preferred style by calling `preferredStatusBarStyle` whenever a status bar update is triggered. In the case of a container view controller, the container can forward the call to a child view controller by implementing the `childViewControllerForStatusBarStyle:` method and returning the child view controller which should receive the `preferredStatusBarStyle` message. If `childViewControllerForStatusBarStyle:` returns `nil` or is not implemented at all, the container view controller itself is expected to return a preferred status bar style.

This is all well and good, but Apple decided not to implement `childViewControllerForStatusBarStyle:` for UINavigationController and UITabBarController, preventing the view controllers contained in these controllers from determining the preferred status bar style.

However, this can easily be added by creating categories for implementing `childViewControllerForStatusBarStyle:`.

Here is how this can be implemented for UITabBarController:

```objc
@implementation UITabBarController (StatusBarStyle)
- (UIViewController *)childViewControllerForStatusBarStyle {
    return self.selectedViewController;
}
@end
```

And UINavigationController:

```objc
@implementation UINavigationController (StatusBarStyle)
- (UIViewController *)childViewControllerForStatusBarStyle {
    return self.topViewController;
}
@end
```

When you include these categories in your project, your navigation and tab bar controllers will enable their selected and top view controllers to control the status bar style. If you use this approach it is a good idea to call `[self setNeedsStatusBarAppearanceUpdate]` in your view controller's `viewDidAppear:` to make sure that your `preferredStatusBarStyle` method gets called.
