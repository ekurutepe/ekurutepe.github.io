---
title: Parallax Scrolling as a Category on UICollectionView
date: 2015-02-17 00:00 UTC
tags: ios, cocoa, development
---

iOS 7 introduced a layered design language where the system achieved to create the illusion of three dimensionality by moving views on screen with the movement of the device. The parallax on the lock screen is probably the best known example of this effect. Some system views such as alert dialogs use this effect and as a developer you can use it in your own apps with the <code>UIMotionEffect</code> classes. <a href="https://twitter.com/ashfurrow">Ash Furrow</a> wrote a really nice <a href="http://www.teehanlax.com/blog/introduction-to-uimotioneffect/">tutorial</a> on how to accomplish this back then in 2013.

Another place where a similar parallax effect can be quite useful is scroll views where the contents of the cells scroll slightly differently than the main scroll view to give the user the illusion of as if the cells are cut-outs showing an object which sits deeper into the screen. The probably best known example of this effect is images in WhatsApp chats. Below you can see this effect in action in a simple demo app:

[caption id="attachment_478" align="aligncenter" width="240"]<img src="http://www.kurutepe.com/wp-content/uploads/2015/02/parallax.gif" alt="A collection view with image cells with parallax scrolling" width="240" height="429" class="size-full wp-image-478" /> A collection view with image cells with parallax scrolling[/caption]

I recently wanted to implement something similar for a collection view with a custom layout. The most straight forward way to achieve this effect is to implement <code>scrollViewDidScroll:</code> and adjust the contents of the cells accordingly as in this <a href="https://github.com/mayuur/MJParallaxCollectionView">project</a>. Another, more elegant approach is like <a href="https://twitter.com/olebegemann">Ole Begemann</a> described in his <a href="http://oleb.net/blog/2014/05/parallax-scrolling-collectionview/">blog post</a> as part of a <code>UICollectionViewLayout</code>.

Since I was already using a <a href="https://github.com/bryceredd/RFQuiltLayout">custom quilt layout</a>, I decided to subclass and extend it with parallax scrolling as Ole described.

Which worked great, as expected, but only in the simulator…

Since the layout gets invalidated at each scroll event, it needs to be recomputed constantly. And in the case of the quilt layout this turned out to be very expensive: a collection with about 50 photos could not be scrolled at 60fps on an iPhone 6.

This could be resolved by implementing some smart caching of the layout attributes in the layout subclass and returning them only by adjusting their parallax offsets. Depending on the nature of the layout this could introduce significant complexity. Complexity is usually a pretty good sign of our code trying to tell us that we're swimming against the flow.

So, I decided to listen to my code and implement parallax scrolling by implementing <code>scrollViewDidScroll:</code> and without touching the layout attributes. Since the app I'm developing, has multiple collection views full of photos already implemented, I could either introduce a common superclass and implement <code>scrollViewDidScroll:</code> there or go the composition route. Due to its flexibility I chose the composition route and decided to implement parallax scrolling as a category on <code>UICollectionViewController</code>

Here is the header for the <code>ParallaxScroll</code> category:

```objc
//UICollectionViewController+ParallaxScroll.h
@import UIKit;

@protocol UICollectionViewCellParallax <NSObject>

- (void)updateWithParallaxOffset:(CGPoint)offset;

@end

@interface UICollectionViewController (ParallaxScroll)

@end

```

It declares an informal protocol with a single method <code>updateWithParallaxOffset:</code>, which we will later implement in our collection view cells which should have parallax scroll.

And the implementation contains a single method:

```objc
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    NSArray *visibleCells = self.collectionView.visibleCells;
    
    for (UICollectionViewCell *cell in visibleCells) {
        if ([cell respondsToSelector:@selector(updateWithParallaxOffset:)]) {
            CGRect bounds = self.collectionView.bounds;
            CGPoint boundsCenter = CGPointMake(CGRectGetMidX(bounds),
                                               CGRectGetMidY(bounds));
            CGPoint cellCenter = cell.center;
            CGPoint offsetFromCenter = 
                CGPointMake(boundsCenter.x - cellCenter.x,
                            boundsCenter.y - cellCenter.y);
            
            CGSize cellSize = cell.bounds.size;
            CGFloat maxVerticalOffset =
            (bounds.size.height / 2) + (cellSize.height / 2);
            CGFloat scaleFactor = 30. / maxVerticalOffset;
            
            CGPoint parallaxOffset = CGPointMake(0.0, -offsetFromCenter.y * scaleFactor);
            [(id)cell updateWithParallaxOffset:parallaxOffset];
        }
    }
}
```

Since the <code>scrollViewDidScroll:</code> is implemented at the <code>UICollectionViewController</code> level all of its subclasses in our app are going to be able to handle parallax scroll. On the other hand, we can implement <code>scrollViewDidScroll:</code> in the subclasses and still retain the parallax scrolling as long as the child implementations call <code>[super scrollViewDidScroll:]</code>.  The informal protocol <code>UICollectionViewCellParallax</code> is central to how this approach works. Parallax scrolling can be selectively turned on and off by implementing the <code>updateWithParallaxOffset:</code> or not in a <code>UICollectionViewCell</code> subclass. For a simple photo cell, <code>updateWithParallaxOffset:</code> can be as simple as moving the image view accordingly:

```objc
- (void)updateWithParallaxOffset:(CGPoint)offset {
    self.imageViewCenterYConstraint.constant = offset.y;
}
```

By implementing parallax scroll with a category and an informal protocol, we get the best of both approaches: the scrolling stays very smooth because the layout does not get recomputed at every scroll position, which might be expensive, and we do get to cleanly separate parallax scroll from the rest of our view controller code in its separate location.