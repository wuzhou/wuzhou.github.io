---
author: wuzhou
comments: true
date: 2013-06-10 08:30:41+00:00
layout: post
slug: cocos2d-for-iphone-2-1-interface-orientations-setting
title: cocos2d for iPhone 2.1 调整屏幕方向
wordpress_id: 144
categories:
- 编程
tags:
- iOS
---

使用cocos2d for iPhone 2.1开发游戏，在要调整屏幕方向时，需要修改AppDelegate.m文件中的两个方法。

首先，修改 " -(NSUInteger)supportedInterfaceOrientations " 方法。修改这个方法可以设置当前开发项目所支持的屏幕方向。如果，要项目的屏幕方向调整为纵向，就需要把纵向的屏幕方向作为程序所支持的方向作为返回值添加到这个方法中。例如：


    -(NSUInteger)supportedInterfaceOrientations
    {
        // iPhone only
        if( [[UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPhone )
        return UIInterfaceOrientationMaskPortrait;  //支持竖直屏幕方向
        // iPad only
        return UIInterfaceOrientationMaskPortrait;
    }


然后，修改

"- (BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation"方法。例如：




    - (BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation
    {
        // iPhone only
        if( [[UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPhone )
            return UIInterfaceOrientationIsPortrait(interfaceOrientation);//判断设备的方向为纵向则返回YES

    	// iPad only
    	// iPhone only
    	return UIInterfaceOrientationIsPortrait(interfaceOrientation);
    }



这样就把屏幕的方向调整为了纵向。
