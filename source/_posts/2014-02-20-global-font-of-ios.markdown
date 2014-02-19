---
layout: post
title: "iOS 下如何设置全局字体？"
date: 2014-02-20 01:52
comments: true
categories: ['iOS', '技术', '字体']
---

# 背景

`iOS 6`跟 `iOS 7`的字体还是有点不一样的，有时候为了两者的统一，或者，应设计师的要求，界面中所有的 Label，Button 等都用自定义字体，一般来说，我们在初始化的时候就需要不断地添加冗余的代码来设置自己的字体。

{% codeblock lang:objc %}
UILabel *label = [[UILabel alloc] init];
label.font = [UIFont fontWithName:@"myFont"];
...

{% endcodeblock %}

如果你的界面全部是代码实现的，而且项目初期就已经定下统一用什么字体了，这就不是什么难事。但是，试想，如果你的界面是由大量`IB`实现的，而且用的是自定义的字体，在`IB`中选都没法选；或是项目已经完成差不多了，上面要求统一改字体，那该如何是好？

其实利用`objective-c`的动态性就可以轻松搞定。

# Method swizzling

什么是`Method Swizzling`请`Google`之，这里只说明方法：

**注意：** *以下方法只用于全局修改由 `Xib` 加载的界面的 UIButton, UILabel的字体，其他的如UITextField等类似，新建Catogery就好，想修改代码生成的界面，修改 initWithCoder 为 init就好*

{% codeblock lang:objc %}
#import <UIKit/UIKit.h>
#import <objc/runtime.h>

@interface UIButton (myFont) @end
@interface UILabel (myFont) @end

@implementation UIButton (myFont)

+ (void)load
{
    Method imp = class_getInstanceMethod([self class], @selector(initWithCoder:));
    Method myImp = class_getInstanceMethod([self class], @selector(myInitWithCoder:));
    method_exchangeImplementations(imp, myImp);
}

- (id)myInitWithCoder:(NSCoder*)aDecode
{
    [self myInitWithCoder:aDecode];
    if (self) {
        CGFloat fontSize = self.titleLabel.font.pointSize;
        self.titleLabel.font = <# Your Font Here #>;
    }
    return self;
}

@end

@implementation UILabel (myFont)

+ (void)load
{
    Method imp = class_getInstanceMethod([self class], @selector(initWithCoder:));
    Method myImp = class_getInstanceMethod([self class], @selector(myInitWithCoder:));
    method_exchangeImplementations(imp, myImp);
}

- (id)myInitWithCoder:(NSCoder*)aDecode
{
    [self myInitWithCoder:aDecode];
    if (self) {
        CGFloat fontSize = self.font.pointSize;
        self.font = <# Your Font Here #>;
    }
    return self;
}

@end

{% endcodeblock %}