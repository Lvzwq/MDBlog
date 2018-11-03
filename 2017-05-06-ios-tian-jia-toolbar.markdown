---
layout: post
title: "IOS 添加toolBar"
date: 2015-11-04 19:34:33 +0800
comments: true
categories: Dev
tags: [IOS]
---

<!--more-->
`UIToolBar`是`UINavigationController`中的一个`Bar`，默认是隐藏的。设置`UINavigationController`的`toolbarHidden`属性可显示`UIToolBar`

也可以不用直接使用`UINavigationController`，直接新建，如下
```objective-c
- (void)addToolBar{
    self.toolBar = [[UIToolbar alloc] initWithFrame:CGRectMake(0, DeviceHeight - 40, DeviceWidth, 40.0f)];
    UIImage *back = [UIImage imageNamed:@"Left"];
    UIBarButtonItem *item1 = [[UIBarButtonItem alloc] initWithImage:back
                                                              style:UIBarButtonItemStyleBordered
                                                             target:self
                                                             action:@selector(dismissController)];
    item1.tintColor = [UIColor greenColor];
    
    UIImage *star = [UIImage imageNamed:@"Star"];
    UIBarButtonItem *item2 = [[UIBarButtonItem alloc] initWithImage:star
                                                              style:UIBarButtonItemStylePlain
                                                             target:self
                                                             action:@selector(starArticle)];
    item2.tintColor = [UIColor greenColor];
    
    UIImage *share = [UIImage imageNamed:@"Share"];
    UIBarButtonItem *item3 = [[UIBarButtonItem alloc] initWithImage:share
                                                              style:UIBarButtonItemStylePlain
                                                             target:self
                                                             action:@selector(shareArticle)];
    item3.tintColor = [UIColor greenColor];
    
    UIImage *comment = [UIImage imageNamed:@"Bubble"];
    UIBarButtonItem *item4 = [[UIBarButtonItem alloc] initWithImage:comment
                                                              style:UIBarButtonItemStylePlain
                                                             target:self
                                                             action:@selector(commentArticle)];
    item4.tintColor = [UIColor greenColor];
    
    UIImage *more = [UIImage imageNamed:@"Star"];
    UIBarButtonItem *item5 = [[UIBarButtonItem alloc] initWithImage:more
                                                                  style:UIBarButtonItemStylePlain
                                                                 target:self
                                                                 action:@selector(more)];
    item5.tintColor = [UIColor greenColor];
    
    UIBarButtonItem *item = [[UIBarButtonItem alloc]
                              initWithBarButtonSystemItem:UIBarButtonSystemItemFlexibleSpace
                              target:nil
                              action:nil];
    

    [self.toolBar setItems:[NSArray arrayWithObjects:item1, item, item2, item, item3, item, item4, item, item5, nil]];
    [self.view addSubview:self.toolBar];
}

```
加一个`item`是为了均匀分布`toolbar`上的空间。