---
layout: post
title: "UIButton 状态的新理解"
date: 2015-07-06 19:45
comments: true
categories: ['ios', 'xcode', 'iphone']
---

`UIButton` 可能是 **iOS** 开发中使用的最多的控件之一了，它拥有极高的可定制性，通过合理的设置可以满足大部分点击响应的需求。一般认为，一个 `UIButton` 有四种状态，即：

- 正常（Normal）
- 高亮（Highlighted）
- 选中（Selected）
- 禁止（Disabled）

但在实际项目中发现，当用户点在一个选中态的按钮上时，按钮显示的是正常状的样式，这是为什么呢？
<!--more-->

事实上，`UIControlState` 是个 BitMask：

```objective-c
typedef NS_OPTIONS(NSUInteger, UIControlState) {
    UIControlStateNormal       = 0,
    UIControlStateHighlighted  = 1 << 0,                  // used when UIControl isHighlighted is set
    UIControlStateDisabled     = 1 << 1,
    UIControlStateSelected     = 1 << 2,                  // flag usable by app (see below)
    UIControlStateApplication  = 0x00FF0000,              // additional flags available for application use
    UIControlStateReserved     = 0xFF000000               // flags reserved for internal framework use
};
```

3 个有效的 Bit 组合其实可以拥有 8 种状态！

我们可以用代码实际测试一下：

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    self.view.backgroundColor = [UIColor whiteColor];

    CGPoint center = CGPointMake(CGRectGetWidth(self.view.frame) / 2, 60.f);
    UIButton *btn0 = [self getButton];
    btn0.center = center;

    center.y += 60.f;
    UIButton *btn1 = [self getButton];
    btn1.center = center;
    btn1.highlighted = YES;
    btn1.enabled = YES;

    center.y += 60.f;
    UIButton *btn2 = [self getButton];
    btn2.center = center;
    btn2.selected = YES;

    center.y += 60.f;
    UIButton *btn3 = [self getButton];
    btn3.center = center;
    btn3.enabled = NO;

    center.y += 60.f;
    UIButton *btn4 = [self getButton];
    btn4.center = center;
    btn4.selected = YES;
    btn4.enabled = NO;

    center.y += 60.f;
    UIButton *btn5 = [self getButton];
    btn5.center = center;
    btn5.highlighted = YES;
    btn5.enabled = NO;

    center.y += 60.f;
    UIButton *btn6 = [self getButton];
    btn6.center = center;
    btn6.highlighted = YES;
    btn6.selected = YES;
    btn6.enabled = NO;

    center.y += 60.f;
    UIButton *btn7 = [self getButton];
    btn7.center = center;
    btn7.highlighted = YES;
    btn7.selected = YES;
    btn7.enabled = YES;

    [self.view addSubview:btn0];
    [self.view addSubview:btn1];
    [self.view addSubview:btn2];
    [self.view addSubview:btn3];
    [self.view addSubview:btn4];
    [self.view addSubview:btn5];
    [self.view addSubview:btn6];
    [self.view addSubview:btn7];
}

- (UIButton *)getButton
{
    UIButton *btn = [UIButton buttonWithType:UIButtonTypeCustom];
    [btn setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];

    [btn setTitle:@"Normal" forState:UIControlStateNormal];
    [btn setTitle:@"Selected" forState:UIControlStateSelected];
    [btn setTitle:@"Highlighted" forState:UIControlStateHighlighted];
    [btn setTitle:@"Highlighted & Disabled" forState:UIControlStateHighlighted | UIControlStateDisabled];
    [btn setTitle:@"Disabled" forState:UIControlStateDisabled];
    [btn setTitle:@"Selected & Disabled" forState:UIControlStateSelected | UIControlStateDisabled];
    [btn setTitle:@"Selected & Highlighted & Disabled" forState:UIControlStateSelected | UIControlStateHighlighted | UIControlStateDisabled];
    [btn setTitle:@"Selected & Highlighted" forState:UIControlStateSelected | UIControlStateHighlighted];

    btn.bounds = CGRectMake(0, 0, 240, 30);

    return btn;
}
```

实验证明，

- `UIControlStateHighlighted` 跟 `UIControlStateHighlighted | UIControlStateDisabled`
- `UIControlStateSelected | UIControlStateHighlighted` 跟 `UIControlStateSelected | UIControlStateHighlighted | UIControlStateDisabled`
 
的效果是一样的，会相互覆盖（苹果的 Bug？），剩下还有 6 种可能的状态。同时我们还能发现，当一种状态（或组合状态）没有设置时，它会使用正常态的设置。

![UIButton State](/images/uibutton-state.png)

所以，如果你的按钮正常是白色的，选中时是<span style="color: red">红色</span>的，而你想让用户在点击一个选中按钮时让它保持<span style="color: red">红色</span>，应该这么做：

```objective-c
UIButton *btn = [UIButton buttonWithType:UIButtonTypeCustom];
[btn setTitleColor:[UIColor whiteColor] forState:UIControlStateNormal];
[btn setTitleColor:[UIColor redColor] forState:UIControlStateSelected];
[btn setTitleColor:[UIColor redColor] forState:UIControlStateSelected | UIControlStateHighlighted];
```

因为点击选中的按钮其实就是选中态与高亮态的叠加，如果没有设置它就用正常态的白色了。