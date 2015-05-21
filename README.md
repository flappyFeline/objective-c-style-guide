# 洪亮的 Objective-C 规范指南

这份规范指南起源于纽约时报 iOS 团队的代码约定，加入并更改了洪亮个人的编程习惯。

## 介绍

关于这个编程语言的所有规范，如果这里没有写到，那就在苹果的文档里： 

* [Objective-C 编程语言][Introduction_1]
* [Cocoa 基本原理指南][Introduction_2]
* [Cocoa 编码指南][Introduction_3]
* [iOS 应用编程指南][Introduction_4]

[Introduction_1]:http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html

[Introduction_2]:https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/Introduction/Introduction.html

[Introduction_3]:https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html

[Introduction_4]:http://developer.apple.com/library/ios/#documentation/iphone/conceptual/iphoneosprogrammingguide/Introduction/Introduction.html


## 目录

* [点语法](#点语法)
* [间距](#间距)
* [条件判断](#条件判断)
	* [三目运算符](#三目运算符)
* [错误处理](#错误处理)
* [方法](#方法)
* [变量](#变量)
* [命名](#命名)
* [注释](#注释)
* [Init 和 Dealloc](#init-和-dealloc)
* [字面量](#字面量)
* [CGRect 函数](#CGRect-函数)
* [常量](#常量)
* [枚举类型](#枚举类型)
* [位掩码](#位掩码)
* [私有属性](#私有属性)
* [图片命名](#图片命名)
* [布尔](#布尔)
* [单例](#单例)
* [导入](#导入)
* [Xcode 工程](#Xcode-工程)

## 点语法

应该 **始终** 使用点语法来访问或者修改属性，访问其他实例时首选括号。

**推荐：**
```objc
view.backgroundColor = [UIColor orangeColor];
[UIApplication sharedApplication].delegate;
```

**反对：**
```objc
[view setBackgroundColor:UIColor.orangeColor];
UIApplication.sharedApplication.delegate;
```

## 间距

* 一个缩进使用 4 个空格，永远不要使用制表符（tab）缩进。请确保在 Xcode 中设置了此偏好。
* 大括号（`if`/`else`/`switch`/`while` 等等）始终和声明在同一行开始，在新的一行结束。
* 多个分支的语句块（`if-else`/`switch`），在条件之间（`else`/`case`）加入空行，有助于视觉清晰度和代码组织性。
* 方法之间应该正好空一行，特别是赋值语句和功能性语句之间，这有助于视觉清晰度和代码组织性。在方法中的功能块之间应该使用空白分开，但往往可能应该创建一个新的方法。
* 方法定义的大括号独占一行，有利于快速区分方法名和第一行代码实现，更加清晰易读
* `@synthesize` 和 `@dynamic` 在实现中每个都应该占一个新行。

**推荐：**
```objc
- (void)foo
{
    User user = [self getUser];
    
    if (user.isHappy) {
        // Do something

    } else if (user.isCalm) {
        // Do something else

    } else {
        // Do something else
    }
}
```

**反对：**
```objc
- (void)foo {
    User user = [self getUser];
    if (user.isHappy) {
        // Do something
    } else if (user.isCalm) {
        // Do something else
    }
    // Do something else
    else {
    }
}
```


## 条件判断

条件判断主体部分应该始终使用大括号括住来防止[出错][Conditionals_1]，即使它可以不用大括号（例如它只需要一行）。这些错误包括添加第二行（代码）并希望它是 if 语句的一部分时。还有另外一种[更危险的][Conditionals_2]，当 if 语句里面的一行被注释掉，下一行就会在不经意间成为了这个 if 语句的一部分。此外，这种风格也更符合所有其他的条件判断，因此也更容易检查。

**推荐：**
```objc
if (!error) {
    return success;
}
```

**反对：**
```objc
if (!error)
    return success;
```

或

```objc
if (!error) return success;
```


[Conditionals_1]:https://github.com/NYTimes/objective-c-style-guide/issues/26#issuecomment-22074256
[Conditionals_2]:http://programmers.stackexchange.com/a/16530

### 三目运算符

三目运算符，? ，只有当它可以增加代码清晰度或整洁时才使用，且使用时应该在运算符前后使用一个空格来提升代码可读性。单一的条件都应该优先考虑使用。多条件时通常使用 if 语句会更易懂，或者重构为实例变量。

**推荐：**
```objc
result = a > b ? x : y;
```

**反对：**
```objc
result = a?x:y;
result = a > b ? x = c > d ? c : d : y;
```

## 错误处理

当引用一个返回错误参数（error parameter）的方法时，应该优先判断该方法本身的返回值，返回值指出方法错误后，再根据错误变量来判断具体是哪一种错误，不得忽略方法本身的返回值而直接判断错误变量，因为一些苹果的 API 在成功的情况下会写一些垃圾值给错误参数（如果非空），所以针对错误变量可能会造成虚假结果（以及接下来的崩溃）。

**推荐：**
```objc
NSError *error;

if (![self trySomethingWithError:&error]) {
    // 处理错误
}
```

**反对：**
```objc
NSError *error;

[self trySomethingWithError:&error];

if (error) {
    // 处理错误
}
```

## 方法

在方法签名中，在 -/+ 符号后应该有一个空格。方法片段之间也应该有一个空格。

**推荐：**
```objc
- (void)setExampleText:(NSString *)text image:(UIImage *)image;
```

## 变量

星号表示指针属于变量，例如：`NSString *text` 不要写成 `NSString* text` 或者 `NSString * text` ，**常量除外**。

尽量定义属性来代替直接使用实例变量。除了初始化方法（`init`， `initWithCoder:`，等）， `dealloc` 方法和自定义的 setters 和 getters 内部，应避免直接访问实例变量，更何况操作一大堆以下划线开头的变量看起来也不具代码美感。更多有关在初始化方法和 dealloc 方法中使用访问器方法的信息，参见[这里][Variables_1]。


**推荐：**

```objc
@interface Section: NSObject

@property (strong, nonatomic) NSString *headline;

@property (strong, nonatomic, readonly) NSString *subtitle;

@end
```

**反对：**

```objc
@interface Section : NSObject {
    NSString *headline;
    NSString *subtitle;
}
```

[Variables_1]:https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW6

#### 变量限定符

当涉及到[在 ARC 中被引入][Variable_Qualifiers_1]变量限定符时，
限定符 (`__strong`, `__weak`, `__unsafe_unretained`, `__autoreleasing`) 应该位于星号和变量名之间，如：`NSString * __weak text`。

[Variable_Qualifiers_1]:(https://developer.apple.com/library/ios/releasenotes/objectivec/rn-transitioningtoarc/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226-CH1-SW4)

## 命名

变量命名应该是有意义的，能够快速、清晰、准确、排除多义性地表达出这个变量本身是什么以及它的用途。

**举例：**

* `NSString *title`：清晰地表达出这是一个标题字符串
* `NSString *titleHTML`：清晰地表达出这不仅是一个标题字符串，而且里面的内容含有HTML标签。尾缀加“HTML”是必要的，这样后续接手的程序员一下子就会明白这个字符串需要使用浏览器解析后才能正确显示出来
* `NSAttributedString *titleAttributedString`：清晰地表达出这不仅是一个标题字符串，而且里面的内容需要使用Attributed Label才能正确显示出来，道理同上
* `NSURL *URL` vs. `NSString *URLString`：与其因为变量名本身容易导致变量类型的不确定性，还不如直接就把变量类型使用在变量名字中以消除歧义。
* `NSDate *lastModifiedDate`：比如这里如果不加尾缀“Date”，仅仅是“lastModified”不能一下子让人确定这是个什么类型的变量（比如也许是时间戳）
* `NSString *releaseDateString`：比如这里如果不加尾缀“String”，那就会给人误解这是一个NSDate类型的变量

变量名应该尽可能命名为描述性的。除了 `for()` 循环外，其他情况都应该避免使用单字母的变量名。

尽可能遵守苹果的命名约定，尤其那些涉及到[内存管理规则][Naming_1]，（[NARC][Naming_2]）的。

尽量不使用缩写，单词完整的名字虽然更长，但却更利于代码的视觉清晰、可读性，更加利于其他人理解。除非是以下这种已经是生活常识的单词缩写，否则必须严格禁止使用缩写：

* HTML
* USB
* fps

**一些iOS特有的缩写是允许的：**

* dict (dictionary，完整写确实太长了，且dict本身也是dictionary的常识性缩写)
* kvo (key-value observing)
* _TODO_

**一些同样是iOS特有的缩写，但我个人强烈不建议使用：**

* vc (viewController)
* _TODO_

**非常不好的缩写：**

* idx (index)
* cnt (count)
* len (length)
* btn (button)
* ctx (context)

为了代码清晰易读易懂，类名、变量、常量应该尽量使用普通单词，必要时加入公司、团队、项目名称的前缀，但字母不超过3个（例如 `NYT`）。

常量应该使用相关类的名字作为前缀并使用全大写+下划线的命名法，而不是驼峰命名法，下划线尽量不超过2个，做到简洁明了。

**推荐：**

```objc
UIButton *settingButton;
```

**反对：**

```objc
UIButton *setBut;
UIButton *setBtn;
```

**推荐：**

```objc
static const NSTimeInterval FADE_DURATION = 0.3;
```

**反对：**

```objc
static const NSTimeInterval fadeTime = 1.7;
```

属性和局部变量应该使用驼峰命名法并且首字母小写。

为了保持一致，实例变量应该使用驼峰命名法命名，并且首字母小写，**并且不加下划线前缀**。以下划线开头的变量名非常影响代码的视觉美感，更何况Xcode的语法高亮能够非常明显地区分出本地变量和实例变量，根本不会存在误会，除非团队中有程序员是色盲。并且，操作属性也应使用 `self.xxx` 的方式，而不要直接操作下划线开头的实例变量，由此带来的额外消耗完全可以忽略不计，但是代码美感却大大提升。

**推荐：**

```objc
@interface Section : NSObject {
    NSString *subtitle;
}

@property (strong, nonatmoic) NSString *headline;

@end

@implementation Section

- (void)foo
{
  self.headline = @"Headline";
  subtitle      = @"Subtitle";
}

@end
```

**反对：**

```objc
@interface Section : NSObject {
    NSString *_subtitle;
}

@property (strong, nonatmoic) NSString *headline;

@end

@implementation Section

- (void)foo
{
  _headline = @"Headline";
  _subtitle = @"Subtitle";
}

@end
```

[Naming_1]:https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html

[Naming_2]:http://stackoverflow.com/a/2865194/340508

## 注释

**注释一定要写中文！**
**注释一定要写中文！**
**注释一定要写中文！**
重要的事说三遍（国际化开发团队除外）。这个理由非常简单：

* 英语不是我们的母语，不能让写注释成为程序员的心智负担
* 98%的开发团队都是本土化的，剩下的1%会慢慢妥协的，最后那1%才是真正的国际化开发团队

**注释尽量单独占一行，而不是跟在代码后面，且//符号后面跟一个空格，中英文混合输入时英文前后不要再加空格**，这样读起来更加清晰

注释应该被用来解释 **为什么** 特定代码做了某些事情，并且当遇到复杂的事情时必须能够讲清楚最终是怎么做到的。所使用的任何注释都必须保持最新，否则就该删除掉。

通常应该避免一大块注释，代码就应该尽量作为自身的文档，只需要隔几行写几句说明。这并不适用于那些用来生成文档的注释。

**推荐：**

```objc
// 防止字幕太短滚动不到上边缘
if (currentY - scrollViewHeight < y) {
    currentY = scrollViewHeight + y;
}
```

**反对：**

```objc
if (currentY - scrollViewHeight < y) {
    currentY = scrollViewHeight + y; //prevent animation can't reach top
}
```


## init 和 dealloc

`dealloc` 方法应该紧跟在 `init` 方法的下面。

`init` 方法的结构应该像这样：

```objc
- (instancetype)init
{
    self = [super init];

    if (self) {
        // ...
    }

    return self;
}
```

## 字面量

每当创建 `NSString`， `NSDictionary`， `NSArray`，和 `NSNumber` 类的不可变实例时，都应该注意英文单词的选择，我们的母语不是英语，应该尽可能使用简单的英语单词，把英语对大脑思考力的消耗维持在尽量低的水平，不使用复数和时态。

* NSDictionary变量以Dict结尾，不使用复式形式
* NSArray变量以Array结尾，不使用复式形式。
* 布尔型变量使用 `is/should` 开头
* 数值型变量使用 `Count/Sum` 这样的单词结尾
* 表达即将发生的事情使用名词加 `Will` 加动词结尾，比如 `WillHappen`
* 表达正在发生的事情使用名词加 `Is` 加动词的正在进行时态，比如 `IsHappening`
* 表达已经发生的事情使用名词加 `DidHappen` 加动词结尾，比如 `DidHappen`

多个变量赋值时每个变量独占一行，并且使用 Xcode 的 [XAlign][XAlign] 插件将等号对齐，大大提升代码美感和可读性。

[XAlign]:http://qfi.sh/XAlign/

**推荐：**

```objc
NSArray *nameArray         = @[ @"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul" ];
NSDictionary *productDict  = @{ @"iPhone" : @"Kate", @"iPad" : @"Kamal", @"Mobile Web" : @"Bill" };
NSNumber *shouldUseLiteral = @YES;
NSNumber *buildingZIPCode  = @10018;
bool isAvailable           = YES;

- (void)dictWillLoad;
- (void)dictIsLoading;
- (void)dictDidLoad;
```

**反对：**

```objc
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
NSDictionary *productManagers = @{@"iPhone" : @"Kate", @"iPad" : @"Kamal", @"Mobile Web" : @"Bill"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingZIPCode = @10018;
bool available = YES;

- (void)dictLoading;
- (void)dictLoaded;
- (void)beforeDictLoad;
- (void)afterDictLoad;
```

## CGRect 函数

当访问一个 `CGRect` 的 `x`， `y`， `width`， `height` 时，应该使用[`CGGeometry` 函数][CGRect-Functions_1]代替直接访问结构体成员。苹果的 `CGGeometry` 参考中说到：

> All functions described in this reference that take CGRect data structures as inputs implicitly standardize those rectangles before calculating their results. For this reason, your applications should avoid directly reading and writing the data stored in the CGRect data structure. Instead, use the functions described here to manipulate rectangles and to retrieve their characteristics.

**推荐：**

```objc
CGRect frame   = self.view.frame;
CGFloat x      = CGRectGetMinX(frame);
CGFloat y      = CGRectGetMinY(frame);
CGFloat width  = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
```

**反对：**

```objc
CGRect frame = self.view.frame;

CGFloat x = frame.origin.x;
CGFloat y = frame.origin.y;
CGFloat width = frame.size.width;
CGFloat height = frame.size.height;
```

[CGRect-Functions_1]:http://developer.apple.com/library/ios/#documentation/graphicsimaging/reference/CGGeometry/Reference/reference.html

## 常量

常量首选内联字符串字面量或数字，因为常量可以轻易重用并且可以快速改变而不需要查找和替换。常量应该声明为 `static` 常量而不是 `#define` ，除非非常明确地要当做宏来使用。

**推荐：**

```objc
static NSString * const COMPANY_NAME  = @"Company";
static const CGFloat THUMBNAIL_HEIGHT = 50.0;
```

**反对：**

```objc
#define CompanyName @"Company"
#define thumbnailHeight 2
```

## 枚举类型

当使用 `enum` 时，建议使用新的基础类型规范，因为它具有更强的类型检查和代码补全功能。现在 SDK 包含了一个宏来鼓励使用使用新的基础类型 - `NS_ENUM()`

**推荐：**

```objc
typedef NS_ENUM(NSInteger, AdRequestState) {
    AdRequestStateInactive,
    AdRequestStateLoading
};
```

## 位掩码

当用到位掩码时，使用 `NS_OPTIONS` 宏。

**举例：**

```objc
typedef NS_OPTIONS(NSUInteger, AdCategory) {
    AdCategoryAutos      = 1 << 0,
    AdCategoryJobs       = 1 << 1,
    AdCategoryRealState  = 1 << 2,
    AdCategoryTechnology = 1 << 3
};
```


## 私有属性

私有属性应该声明在类实现文件的延展（匿名的类目）中。

**推荐：**

```objc
@interface NYTAdvertisement ()

@property (nonatomic, strong) GADBannerView *googleAdView;
@property (nonatomic, strong) ADBannerView  *iAdView;
@property (nonatomic, strong) UIWebView     *adXWebView;

@end
```

## 图片命名

图片等资源文件的名称统一使用全小写+下划线，这样无论在任何系统中都可以让设计师不去关注大小写敏感问题。图片所在的目录也使用全小写+下划线的形式。

**推荐：**

* `logo_chs.png`
* `common_box.png`
* `background_1.jpg`


## 布尔

因为 `nil` 解析为 `NO`，所以没有必要在条件中与它进行比较。永远不要直接和 `YES` 进行比较，因为 `YES` 被定义为 1，而 `BOOL` 可以多达 8 位。

这使得整个文件有更多的一致性和更大的视觉清晰度。

**推荐：**

```objc
if (!someObject) {
}
```

**反对：**

```objc
if (someObject == nil) {
}
```

_如果Objective-C代码有被移植到Java的需要，则以上原则不适用，需要反过来做，因为Java中比较变量是否为null必须写双等号_

-----

**对于 `BOOL` 来说, 这有两种用法:**

```objc
if (isAwesome)
if (![someObject boolValue])
```

**反对：**

```objc
if ([someObject boolValue] == NO)
if (isAwesome == YES) // 永远别这么做
```

-----

如果一个 `BOOL` 属性名称是一个形容词，属性可以省略 “is” 前缀，但为 get 访问器指定一个惯用的名字，例如：

```objc
@property (assign, getter=isEditable) BOOL editable;
```

内容和例子来自 [Cocoa 命名指南][Booleans_1] 。

[Booleans_1]:https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE


## 单例

单例对象应该使用线程安全的模式创建共享的实例。

```objc
+ (instancetype)sharedInstance {
   static id sharedInstance = nil;

   static dispatch_once_t onceToken;
   dispatch_once(&onceToken, ^{
      sharedInstance = [[self alloc] init];
   });

   return sharedInstance;
}
```
这将会预防[有时可能产生的许多崩溃][Singletons_1]。

[Singletons_1]:http://cocoasamurai.blogspot.com/2011/04/singletons-your-doing-them-wrong.html

## 导入   

如果有一个以上的 import 语句，就对这些语句进行[分组][Import_1]。每个分组的注释是可选的。   
注：对于模块使用 [@import][Import_2] 语法。   

```objc   
// Frameworks
@import QuartzCore;

// Models
#import "User.h"

// Views
#import "Button.h"
#import "UserView.h"
```   


[Import_1]: http://ashfurrow.com/blog/structuring-modern-objective-c
[Import_2]: http://clang.llvm.org/docs/Modules.html#using-modules


## Xcode 工程

为了避免文件杂乱，物理文件应该保持和 Xcode 项目文件同步。Xcode 创建的任何组（group）都必须在文件系统有相应的映射。为了更清晰，代码不仅应该按照类型进行分组，也可以根据功能进行分组。


如果可以的话，尽可能一直打开 target Build Settings 中 "Treat Warnings as Errors" 以及一些[额外的警告][Xcode-project_1]。如果你需要忽略指定的警告,使用 [Clang 的编译特性][Xcode-project_2] 。


[Xcode-project_1]:http://boredzo.org/blog/archives/2009-11-07/warnings

[Xcode-project_2]:http://clang.llvm.org/docs/UsersManual.html#controlling-diagnostics-via-pragmas

