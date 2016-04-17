---
layout: post
title: "如何优雅地编写 iOS 第三方库"
date: 2016-04-17 10:00
comments: true
categories: ['iOS', '三方库', '优雅', '接口']
---

**iOS** 经过八年多的发展，已经涌现出诸多优秀的第三方库，但怎样才算是优雅？总体来说，[AFNetworking](https://github.com/AFNetworking/AFNetworking) 就十分优雅，而 [GPUImage](https://github.com/BradLarson/GPUImage) 就只是可用，而不算优雅。编写优雅的第三方库，就像制作一件精美的艺术品一样，过程让人沉醉，结果令人赏心悦目。否则就是单纯的代码堆积与功能实现，过程像搬砖，完成后也没有成就感。下面就本人的一点经验，分享下如何优雅地封装第三方库。

<!--more-->

## 命名

好的命名规则是一个成功的第三库的开始，然而现实中很许多人随性命名，导致沟通成本上升。事实上命名问题上，苹果有其官方统一的标准，即首字母小写的驼峰式，如：`setName:`，`reloadDataWithName:andEmail:` 等，且一般约定在 **getter** 不使用 `getName`，而直接使用 `name`。

### 属性

属性的命名最好是能直接表达其意义的英文名词，当然合适的时候添加形容词，如：

```obj-c
@interface XTUser : NSObject
@property (nonatomic, strong) NSString *fullName;
@property (nonatomic, strong) NSString *firstName, *lastName;
@property (nonatomic, strong) NSString *phoneNumber;
@property (nonatomic, strong) NSString *email;
@end
```

如果是数组或集合等，用名词的复数形式：

```obj-c
@interface XTDownloadManager : NSObject
@property (nonatomic, strong) NSArray<NSURL *> *downloadURLs;
@end
```

表达数量的属性，可以加 `numberOfXXX`，或 `XXXCount` 如：

```obj-c
@interface XTBook : NSObject
@property (nonatomic, assign) NSInteger numberOfPages;
@property (nonatomic, assign) NSInteger pageCount;
@end
```

表示对象状态的属性，使用英文的**正在进行时**，或**完成时**等可以表示状态的词，如：

- `@property (nonatomic, assign) BOOL isClosed;` ，已经关闭
- `@property (nonatomic, assing) BOOL isClosing;` ，正在关闭
- `@property (nonatomic, assing) BOOL isAvaliable;`，目前可用
- `@property (nonatomic, assign) BOOL hasChanged;`，已经被改变

而以下写法表意是不明确的：

- ~~`@property (nonatomic, assing) BOOL isClose;`~~

当然，前面的代码还能有更优雅的写法：

- `@property (nonatomic, assign, getter=isClosed) BOOL closed;`
- `@property (nonatomic, assing, getter=isClosing) BOOL closing;`
- `@property (nonatomic, assing, getter=isAvaliable) BOOL avaliable;`
- `@property (nonatomic, assign, getter=hasChanged) BOOL changed;`

### 枚举

**iOS** 基础类库有着比其它任何官方库都好用的枚举类型，因为在使用过程中没有任何痛苦，也无需查文档，照着它的类型打完然后就有表意明确的自动补全，如 `UIKit` 中很常用的：

```obj-c
typedef NS_ENUM(NSInteger, UIControlContentHorizontalAlignment) {
    UIControlContentHorizontalAlignmentCenter = 0,
    UIControlContentHorizontalAlignmentLeft   = 1,
    UIControlContentHorizontalAlignmentRight  = 2,
    UIControlContentHorizontalAlignmentFill   = 3,
};

typedef NS_OPTIONS(NSUInteger, UIControlState) {
    UIControlStateNormal       = 0,
    UIControlStateHighlighted  = 1 << 0,                  // used when UIControl isHighlighted is set
    UIControlStateDisabled     = 1 << 1,
    UIControlStateSelected     = 1 << 2,                  // flag usable by app (see below)
    UIControlStateFocused NS_ENUM_AVAILABLE_IOS(9_0) = 1 << 3, // Applicable only when the screen supports focus
    UIControlStateApplication  = 0x00FF0000,              // additional flags available for application use
    UIControlStateReserved     = 0xFF000000               // flags reserved for internal framework use
};
```

等等。它的特点也很明显，就是

```c
enum 类型名 {
  类型名+枚举名0 = 枚举值0,
  类型名+枚举名1 = 枚举值1,
  类型名+枚举名2 = 枚举值2,
}
```

于是有一段时间，本人都想把 **OpenGL** 里那些恶心的宏重写成这样形式（<https://github.com/rickytan/Cocoa-Style-OpenGL>），如：

```c
CC_ENUM(int, GLUnsignedType) {
	GLUnsignedTypeByte  = GL_UNSIGNED_BYTE,
	GLUnsignedTypeShort = GL_UNSIGNED_SHORT,
	GLUnsignedTypeInt   = GL_UNSIGNED_INT,
};

CC_ENUM(int, GLDrawMode) {
	GLDrawModePoints        = GL_POINTS,
	GLDrawModeLines         = GL_LINES,
	GLDrawModeLineLoop      = GL_LINE_LOOP,
	GLDrawModeLineStrip     = GL_LINE_STRIP,
	GLDrawModeTriangles     = GL_TRIANGLES,
	GLDrawModeTriangleStrip = GL_TRIANGLE_STRIP,
	GLDrawModeTriangleFan   = GL_TRIANGLE_FAN,
	GLDrawModeQuads         = GL_QUADS,
	GLDrawModeQuadStrip     = GL_QUAD_STRIP,
	GLDrawModePolygon       = GL_POLYGON,
};
```

然而苦于功底有限，同时对 **OpenGL** 底层了解也不够，作罢。

而当前市面上已有的部分厂商并没有按照这个约定来发布 SDK，以致于要不断查看文档才知道如何操作。如一直倍受诟病的鹅厂：

```obj-c
enum QQApiInterfaceReqType
{
    EGETMESSAGEFROMQQREQTYPE = 0,   ///< 手Q -> 第三方应用，请求第三方应用向手Q发送消息
    ESENDMESSAGETOQQREQTYPE = 1,    ///< 第三方应用 -> 手Q，第三方应用向手Q分享消息
    ESHOWMESSAGEFROMQQREQTYPE = 2   ///< 手Q -> 第三方应用，请求第三方应用展现消息中的数据
};

/**
 QQApi请求消息基类
 */
@interface QQBaseReq : NSObject

/** 请求消息类型，参见\ref QQApiInterfaceReqType */
@property (nonatomic, assign) int type;

@end
```

以上 **SDK** 有三个问题：

1. `QQBaseRequest` 不需要缩写为 `QQBaseReq`，在 **Objective-C** 的世界，名字再长都不过份，而要能将类、方法等功能表述清楚；
2. `type` 应当明确指定类型，就一个 `int` 让使用者不知所措；
3. `QQApiInterfaceReqType` 的枚举名应当以 QQApiInterfaceReqType 开头，且以驼峰式命名，方便自动补全。全大写一般是宏定义。

## 接口及实现

在编写第三方库时，尽量面向接口编程，并给一个默认的实现，方便使用者扩展。

例如，你实现了一个照片显示的 **View**，你的类定义如下：

```obj-c
@interface MyPhoto : NSObject
@property (nonatomic, strong) UIImage *thumbnailImage;
@property (nonatomic, strong) NSString *name;
@property (nonatomic, strong) NSURL *originURL;
@end

@interface MyPhotoGalleryView : UIView
@property (nonatomic, strong) NSArray<MyPhoto *> *photos;
@end
```

然而在实际项目中，使用者一般会有自定义的照片对象（如：`@class XTPhoto`），为了使用你的实现，他不得不将他的对象转成你的，再设置到 `photos` 属性，造成不必要的内存浪费。

```obj-c
@interface XTPhoto : NSObject
@property (nonatomic, strong) UIImage *image;
@property (nonatomic, strong) NSString *photoPath;
@end

...

NSMutableArray *photos = [NSMutableArray arrayWithCapacity:self.photoGallery.count];
for (XTPhoto *photo in self.photoGallery) {
    MyPhoto *my = ...;
    [photos addObject:my];
}
photoGalleryView.photos = [NSArray arrayWithArray:photos];
```

而如果换一种实现，定义一个接口，就可以一定程度上避免这种问题：

```obj-c
@protocol MyPhoto <NSObject>
@required
@property (nonatomic, readonly) UIImage *thumbnailImage;
@property (nonatomic, readonly) NSString *name;

@optional
@property (nonatomic, readonly) NSURL *originURL;

@end

@interface MyPhoto : NSObject <MyPhoto>
@property (nonatomic, strong) UIImage *thumbnailImage;
@property (nonatomic, strong) NSString *name;
@property (nonatomic, strong) NSURL *originURL;
@end

@interface MyPhotoGalleryView : UIView
@property (nonatomic, strong) NSArray<id<MyPhoto> > *photos;
@end

@interface XTPhoto : NSObject <MyPhoto>
@property (nonatomic, strong) UIImage *image;
@property (nonatomic, strong) NSString *photoPath;
@end

@implementation XTPhoto

- (UIImage *)thumbnailImage
{
    return self.image;
}

- (NSString *)name
{
    return self.photoPath.lastPathComponent;
}

- (NSURL *)originURL
{
    return [NSURL URLWithString:self.photoPath];
}
@end
```

这样可以直接将 `self.photoGallery` 直接赋值到 `photoGalleryView.photos`。



除了面向接口编程，在类的实现上也应遵循以下原则：

* 保持简单。只向外暴露能实现功能的最少的接口。假如你有一个视图，里面有一个 Label 显示了剩余的金币数，大于 0 时为红色，小于 0 时为绿色，那么应当只暴露一个 **NSInteger** 接口。

        @interface XTStatsView : UIView
        @property (nonatomic, assign) NSInteger numberOfGold;
        @end

    然后在具体的实现中设置 Label 的值：

        @interface XTStatsView ()
        @property (nonatomic, strong) UILabel *goldLabel;
        @end

        @implementation XTStatsView
        - (void)setNumberOfGold:(NSInteger)numberOfGold
        {
            _numberOfGold = numberOfGold;
            self.goldLabel.text = [NSString stringWithFormat:@"%d 金币", _numberOfGold];
            if (_numberOfGold >= 0) {
                self.goldLabel.textColor = [UIColor redColor];
            }
            else {
                self.goldLabel.textColor = [UIColor greenColor];
            }
        }
        @end

    一种比较懒的办法是直接暴露 `goldLabel`，给了使用者较多的灵活性，但也容易造成一些不可预料的结果。

    另外有一些情况属于暴露多余接口，如你自定义了一个 Cell，用来展示某个 Entity 的内容，于是定义如下：

        @interface MyCell: UITableViewCell
        @property (nonatomic, strong) MyEntity *entity;
        - (void)renderData;
        @end

    使用者设置了 Entity 后还要调用 `- (void)renderData` 才能显示出来，这其实是多余的，可以去掉 `- (void)renderData` 而在 Entity 的 **setter** 中内部调用 `- (void)renderData` 或其他类似的方法。使用者所设置的，即是所看到（所得到）的，不需要再调用其他方法。

* 调用顺序无关。如果你实现的类有较多的状态无关的属性，它应该是调用顺序无关的。例如，一个 **VC** 暴露了一个 `titleColor` 属性，可以设置视图中 Label 的颜色，一个比较常见的错误是如下实现：

        @interface MyViewController: UIViewController
        @property (nonatomic, strong) UIColor *titleColor;
        @end

        @implementation MyViewController
        - (void)viewDidLoad
        {
            [super viewDidLoad];
            self.titleLabel.textColor = self.titleColor;
        }
        @end

    以上实现的问题在于，如果 **VC** 的视图已经加载，那么设置 `titleColor` 将无效。正确的实现应该如下：

        @implementation MyViewController
        - (void)viewDidLoad
        {
            [super viewDidLoad];
            self.titleLabel.textColor = self.titleColor;
        }

        - (void)setTitleColor:(UIColor *)titleColor
        {
            if (_titleColor != titleColor) {
                _titleColor = titleColor;
                if (self.isViewLoaded) {
                    self.titleLabel.textColor = self.titleColor;
                }
            }
        }
        @end

    这样保证使用者无论何时设置都是有效的。

## 宏

合理地使用 **NS** 自带的编译器预处理宏定义可以马上提高整个代码的逼格，如：

```obj-c
UIKIT_EXTERN NSString *const MyUserDidLoginNotification;

@interface MyUser: NSObject
@property (nonatomic, strong) NSString *name DEPRECATED_MSG_ATTRIBUTE("Use userName instead!");
@property (nonatomic, strong) NSString *userName;

- (instancetype)initWithName:(NSString *)name email:(NSString *)email NS_DESIGNATED_INITIALIZER;
- (void)reloadData NS_REQUIRES_SUPER;
@end

@interface MyUserManager : NSObject
+ (instancetype)sharedManager;

- (instancetype)init NS_UNAVAILABLE;

@end
```

更多宏请参见 `<Foundation/NSObjCRuntime.h>`。
