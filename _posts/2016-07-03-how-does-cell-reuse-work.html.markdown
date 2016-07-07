---
title: How Does Cell Reuse Work
date: 2016-07-03 20:59 CEST
tags: ios, cocoa, development
published: false
---

`UITableView` and `UICollectionView` are such workhorses on iOS, that it's hard to think of a non-game app, which does not use one of these classes. One of the amazing features of the original iPhone back in 2007 was how smoothly the tables scrolled on the screen. Given the very limited processing power back then, this was only possible, among other really smart tricks, by not allocating cells for each row scrolling into the screen but by reusing them after they have scrolled off the screen. This year's [WWDC video on collection views](https://developer.apple.com/videos/play/wwdc2016/219/) is a must-watch to learn how this is still an important factor you have to take into account  to achieve 60fps while scrolling and also how iOS 10 helps you in this regard. This post is not going to be about how to take advantage of cell reuse in your app but consider some trade-offs it involves and explore how it could be implemented from scratch. 


