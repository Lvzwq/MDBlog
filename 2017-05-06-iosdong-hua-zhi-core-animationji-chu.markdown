---
layout: post
title: "IOS动画之Core Animation基础"
date: 2015-12-17 23:08:04 +0800
comments: true
categories: Dev
tags: [IOS]
---


<!--more-->
首先附上一张从网上找到的`Core Animation`的结构图。
![Core Animaion](http://zhangwenqiang.com.cn/zwq/core_annimation.jpg)

**准备**: 使用`Core Animation`制作动画需要引入`QuartzCore.framework`库，并在制作动画的地方引入`#import <QuartzCore/QuartzCore.h>`

### `CABasicAnimation`(关键帧动画)
所谓关键帧动画，就是将`Layer`的属性作为`KeyPath`来注册，指定动画的起始帧和结束帧，然后自动计算和实现中间的过渡动画的一种动画方式。
```objective-c
CABasicAnimation *animation = [CABasicAnimation animation];
animation.keyPath = @"position.x";
animation.duration = 1;
animation.fromValue = @77;
animation.toValue = @455;
animation.repeatCount = 1;
animation.beginTime = CACurrentMediaTime() + 0.5;
animation.autoreverses = YES;
[layer addAnimation:animation forKey:@"basic"];
```
动画属性参数说明:

- `keyPath`：动画的运动属性。

常见的支持有: [见官方文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Key-ValueCodingExtensions/Key-ValueCodingExtensions.html)
> 平移:
- `position`(点到点坐标)  
- `position.x`或者`transform.translation.x`(x坐标方向的移动)
- `position.y`或者`transform.translation.y`(y坐标方向的移动)
缩放:`scale`
旋转:`rotation`
- `transform.rotation.x`(绕x坐标轴旋转)
- `transform.rotation.y`(绕y坐标轴旋转)
```
animation.keyPath = @"position";
animation.keyPath = @"transform.scale"
animation.keyPath = @"transform.rotation.y"
```

- `duration`： 动画持续时间,以秒为单位
- `fromValue`： 动画的开始状态(绝对定位)
- `toValue`：动画的结束状态(绝对定位)
- `byValue`：动画的相对结束位置(相对定位)。

动画中`fromValue` -> `toValue`表示从开始位置到结束位置(绝对位置)，也可以设定从当前位置到加上`byValue`(相对位置)。注意 `fromValue`、`toValue`和`byValue`是`id`类型，如果是基本类型的话，需要转化为OC对应的类类型@。
```
animation.fromValue = [NSNumber numberWithFloat:50.0];
animation.toValue = @500.0;
animation.byValue = [NSValue valueWithCGPoint:CGPointMake(300, 300)];
```
- `repeatCount`:表示动画重复次数。默认为0，指定为`HUGE_VALF`， 永久执行.
- `autoreverses`:动画结束之后是否执行逆动画。
- `beginTime`:指定动画执行的时间。如果需要动画延迟，可以加上`CACurrentMediaTime()+ 延迟的秒数`。
- `timingFunction`: 设定动画的速度变化。
```
animation.timingFunction =  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseInEaseOut]; //先加速后减速
```

######  动画结束之后，动画界面又恢复动画前的状态。要想保持动画后的状态，可以有以下两种方式
**方法一(推荐)**
这种方式有局限性，适用于消失效果的动画。
```
layer.position = CGPointMake(455, 61);
```
**方法二(通用)**
可以添加以下两行代码
```
animation.fillMode = kCAFillModeForward;
animation.removedOnCompletion = NO;
```
默认动画执行结束之后，就从渲染树`render tree`中移除,设置`removedOnCompletion`为`NO`,保留渲染后的动画。

### `CAKeyframeAnimation`组合动画
`CABasicAnimation`动画是一次执行单一效果的动画,`CAKeyframeAnimation`可以在一次动作中执行多种效果。

```
//为动画图形运动划定运动路线
//定义动画贝塞尔曲线路径
UIBezierPath *path = [UIBezierPath bezierPath]; 
//定义起点
[path moveToPoint:CGPointMake(50, 150)]; 
//结束点、中间点
[path addQuadCurveToPoint:CGPointMake(270, 300) controlPoint:CGPointMake(150, 20)];
//定义2个路径点
//[path addCurveToPoint:CGPointMake(50, 500) controlPoint1: CGPointMake(350, 400) controlPoint2:CGPointMake(100, 450)];
 
CAKeyframeAnimation *animation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
animation.path = path.CGPath; //设置路径
animation.rotationMode = kCAAnimationRotateAuto;

//放大
CABasicAnimation *expandAnimation = [CABasicAnimation animationWithKeyPath:@"transform.scale"];
expandAnimation.duration = 0.5f;
expandAnimation.fromValue = [NSNumber numberWithFloat:1];
expandAnimation.toValue = [NSNumber numberWithFloat:2.0f];
expandAnimation.timingFunction=[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];

//缩小
CABasicAnimation *narrowAnimation = [CABasicAnimation animationWithKeyPath:@"transform.scale"];
narrowAnimation.beginTime = 0.5;
narrowAnimation.fromValue = [NSNumber numberWithFloat:2.0f];
narrowAnimation.duration = 1.5f;
narrowAnimation.toValue = [NSNumber numberWithFloat:0.5f];
narrowAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut];

//添加动画集
CAAnimationGroup *groups = [CAAnimationGroup animation];
groups.animations = @[animation,expandAnimation,narrowAnimation];
groups.duration = 2.0f;
groups.removedOnCompletion = NO;
groups.fillMode = kCAFillModeForwards;
groups.delegate = self;
[layer addAnimation:groups forKey:@"group"];
```
### 监听动画执行开始和结束的状态
指定委托对象，实现委托方法
```
animation.delegate = self; // 指定委托对象 

#pragma mark CAAnimation Delegate
- (void)animationDidStart:(CAAnimation *)anim{
    NSLog(@"动画启动");
}

- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag{
    NSLog(@"动画结束");
}
```

### IOS 动画效果的开源库
- [RBBAnimation](https://github.com/robb/RBBAnimation)
- [Canvas](https://github.com/CanvasPod/Canvas)
- [Facebook pop](https://github.com/facebook/pop)
- [popping](https://github.com/schneiderandre/popping)


#### 参考:
- [Animations Explained](https://www.objc.io/issues/12-animations/animations-explained/) / [中文翻译](http://objccn.io/issue-12-1/)
- [CSDN博客 - CABasicAnimation的基本使用方法（移动·旋转·放大·缩小）](http://blog.csdn.net/iosevanhuang/article/details/14488239)


#### 深入理解
- [Further Reading](https://www.objc.io/issues/12-animations/animations-explained/#further-reading)
- [UIKit Dynamics](http://www.raywenderlich.com/50197/uikit-dynamics-tutorial)