---
layout: post
title: "使你的Navigation Bar难以置信地兼容iOS6/7"
date: 2013-12-19 15:30:20 +0800
comments: true
categories: 
---

自定义UI控件的外观是iOS开发者经常遇到的问题,再加上烦人的系统兼容,足够你研究一阵子了.

是时候解决这个痛点了! 我们就先从UINavigationBar开始吧~

<!-- more -->

- [iOS中默认的导航栏](#anchor1)
- [设置导航栏的背景颜色](#anchor2)
- [在导航栏中使用背景图片](#anchor3)
- [导航栏与状态栏](#anchor4)
- [定制返回按钮的颜色](#anchor5)
- [修改导航栏标题的字体](#anchor6)
- [修改导航栏标题为图片](#anchor7)
- [添加多个按钮](#anchor8)
- [修改状态栏的风格](#anchor9)
- [隐藏状态栏](#anchor10)
- [接下来?](#anchor11)


##[iOS中默认的导航栏](id:anchor1)

<img src="/images/article/custom_navigation_bar/contrast_default_navigation_bar.png">

##[设置导航栏的背景颜色](id:anchor2)

在iOS 7中，不再使用`tintColor`属性来设置导航栏的颜色，而是使用`barTintColor`属性来修改背景色。我们可以在`AppDelegate.m`文件中 定义方法`- (void)customizeAppearance`里面添加如下代码来修改颜色,并在`didFinishLaunchingWithOptions:`中调用.

首先我们先定义个宏,来帮助我们设置颜色:

``` objc 
#define UIColorFromRGB(rgbValue) \
[UIColor colorWithRed:((float)((rgbValue & 0xFF0000) >> 16))/255.0 \
                green:((float)((rgbValue & 0xFF00) >> 8))/255.0 \
                 blue:((float)(rgbValue & 0xFF))/255.0 alpha:1.0]

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    [self customizeAppearance];
    return YES;
}

- (void)customizeAppearance
{
    [[UINavigationBar appearance] setBarTintColor:UIColorFromRGB(0x0099cc)];
}
```

在iOS6中请使用:

``` objc
[[UINavigationBar appearance] setTintColor:UIColorFromRGB(0x0099cc)];
```

<table>
    <caption><strong>Table 1-1 </strong>Contrast</caption>
    <tr>
        <th></th>
        <th>ios7</th>
        <th>ios6</th>
    </tr>
    <tr>
        <td>Bar style</td>
        <td>Translucent light (default) or translucent dark.</td>
        <td>Opaque gradient blue (default) or opaque black.</td>
    </tr>
    <tr>
        <td>Translucent</td>
        <td>Default YES</td>
        <td>Default NO</td>
    </tr>
    <tr>
        <td>Appearance</td>
        <td>底部有一像素的线</td>
        <td>底部边缘有投影</td>
    </tr>
    <tr>
        <td>Tinting</td>
        <td>
            tintColor 用来设置bar button items <br>
            barTintColor 用来设置bar的背景色
        </td>
        <td>tintColor 用来设置bar的背景色</td>
    </tr>
</table>
<br>


iOS7默认情况下，导航栏的translucent属性为YES.并对导航栏做模糊处理。
你可以在对应的ViewController中设置translucent.

而在iOS6及以前默认为NO,如果barStyle设置为UIBarStyleBlackTranslucent 那么translucent属性将总是YES.
如下图，是translucent值为NO和YES的对比效果：

iOS7:

<img src="/images/article/custom_navigation_bar/ios7_contrast-_translucent_navigation_bar.png" alt="default_navigation_ios7"/>

iOS6:

<img src="/images/article/custom_navigation_bar/ios6_contrast-_translucent_navigation_bar.png" alt="default_navigation_ios7"/>

##[在导航栏中使用背景图片](id:anchor3)

如果想要获得更惊艳的效果,那么你需要提供一张图片作为背景.你可以用以下方法设置背景图片:

``` objc
[[UINavigationBar appearance] setBackgroundImage:[UIImage imageNamed:@"navigation_bar_bg"] forBarMetrics:UIBarMetricsDefault];
```
一般情况下你可能会用这样一张图片(320x44)

<img src="/images/article/custom_navigation_bar/navigation_bar_bg@2x.png" alt="default_navigation_ios7"/ style="width: 200px;">

但实际上我们并不推荐这么使用,因为纹理相同的图片我们可以截取其中一段并平铺,这样做的好处是可以让你的App占用更少的空间,也为将来适配更多尺寸埋下伏笔.

iOS 为我们提供了平铺图片的方法:

``` objc
UIEdgeInsets insets = UIEdgeInsetsMake(0, 10, 0, 10);
UIImage *navigationBGImage = [[UIImage imageNamed:@"navigation_bar_bg_pattern_44"]
                      resizableImageWithCapInsets:insets];
[[UINavigationBar appearance] setBackgroundImage:navigationBGImage forBarMetrics:UIBarMetricsDefault];
```

我们可以选择平铺的范围,而边缘我们可以让图片保留原有的纹理.示意图:

<img src="/images/article/custom_navigation_bar/reszie_image_demo.png" alt="default_navigation_ios7"/ style="width: 200px;">

##[导航栏与状态栏](id:anchor4)

在iOS7中导航栏的高度从44 points(88 pixels)变为了64 points(128 pixels)。
获得的效果也与我们提供的图片高度有关.

你需要注意:

<table border="0" cellspacing="0" cellpadding="5">
        <caption><strong>Table 1-2 </strong>Treatment of resizable background images for bars at the top of the screen</caption>
        <thead>
            <tr>
                <th scope="col"><p>
                    Height
                </p></th>
                <th scope="col"><p>
                    Resizing treatment
                </p></th>
                <th scope="col"><p>
                    Status bar background appearance
                </p></th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td scope="row"><p>
                    44 points
                </p></td>
                <td><p>
                    Horizontally resized as appropriate (the image is not vertically tiled or stretched).
                </p></td>
                <td><p>
                    Black, if using <code class="code-voice">UIBarPositionTopAttached</code>.
                </p><p>
                Provided by the window background, if using <code class="code-voice">UIBarPositionTop</code>.
            </p></td>
        </tr>
        <tr>
            <td scope="row"><p>
                Less than 44 points
            </p></td>
            <td><p>
                Vertically resized to 64 points if using <code class="code-voice">UIBarPositionTopAttached</code> or 44 points if using <code class="code-voice">UIBarPositionTop</code>.
            </p><p>
            Horizontally resized as appropriate.
        </p></td>
        <td><p>
            Provided by the bar background.
        </p></td>
    </tr>
    <tr>
        <td scope="row"><p>
            64 points
        </p></td>
        <td><p>
            Horizontally resized as appropriate.
        </p></td>
        <td><p>
            Provided by the bar background.
        </p></td>
    </tr>
    <tr>
        <td scope="row"><p>
            1 point
        </p></td>
        <td><p>
            Vertically resized to 64 points if using <code class="code-voice">UIBarPositionTopAttached</code> or if using a navigation controller. Vertically resized to 44 points if using <code class="code-voice">UIBarPositionTop</code>.
        </p><p>
        Horizontally resized as appropriate.
    </p></td>
    <td><p>
        Provided by the bar background.
    </p></td>
</tr>
</tbody>
</table>
<br>

简单的说:

如果你想让`Status Bar`保持iOS6的效果(黑色背景)请使用高度为`44points`的图片并只是水平方向平铺.

如果你想让`Status Bar`保持iOS7的效果 请使用高度为`64points`的图片并保持水平方向平铺,或者使用高度为`44points`的图片,水平和垂直方向都进行平铺.

我们来看下效果:

<img src="/images/article/custom_navigation_bar/iOS7_navigation_bar_pattern.png"/>

<img src="/images/article/custom_navigation_bar/iOS6_navigation_bar_pattern.png"/>

我们发现使用64points的图片对iOS6的支持并不好,所以这里还是建议IOS6/7分别用44/64高度的图片.

这里还是需要补充下:

<table>
    <tr>
        <td>iOS7</td>
        <td>如果使用不透明的图片作为背景 并将translucent设为YES(默认) 那么系统会为图片设置一个小于1.0的不透明度 <br>
                若要获得不透明的效果 请将并将translucent设为NO <br>
            如果使用透明的图片作为背景 且将translucent设为NO 系统会提供一个以barTintColor为颜色的不透明的背景 <br>
            若barTintColor为nil的话,会根据bar style来改变背景 UIBarStyleBlack --> 黑色 UIBarStyleDefault -->白色
        </td>
    </tr>
    <tr>
        <td>iOS6</td>
        <td>
            如果使用不透明图片作为背景 并将translucent设为NO(默认) status bar会用图片的主色调作为背景 <br>
            如果使用不透明图片作为背景 并将translucent设为YES     status bar会变为黑色背景 <br>
                此时若不设background color 则系统会为图片设置一个小于1.0的不透明度 <br>
                若设置background color 则navigation bar 不透明 <br>
            如果使用透明图片作为背景 并将translucent设为NO(默认) 系统会为navigation bar 提供一个以tintColor为颜色的背景<br>
                若要获得透明效果 请将translucent设为YES
        </td>
    </tr>
</table>
<br>

Tips:

> iOS6在导航栏下方有系统提供的阴影,你可以替换成自己的图片,或者将它干掉:

``` objc
[[UINavigationBar appearance] setShadowImage: [[UIImage alloc] init]];
```
在iOS7中Status Bar是透明的,我们的视图会与Status bar有所重合.在这里我们需要注意视图的Layout.
在有导航栏的界面我们需要根据不同的情况进行处理:

<table>
    <caption><strong>Table 1-3 </strong>Status bar & Translucent</caption>
    <tr>
        <th>Translucent</th>
        <th>iOS7</th>
        <th>iOS6</th>
    </tr>
    <tr>
        <td>YES</td>
        <td>视图从Status bar <code>顶部</code>算起</td>
        <td>视图从Status bar <code>底部</code>算起</td>
    </tr>
    <tr>
        <td>NO</td>
        <td>视图从Navigation bar <code>底部</code>算起</td>
        <td>视图从Navigation bar <code>底部</code>算起</td>
    </tr>
</table>
<br>

我们来看一下效果, 首先创建一个Label:
``` objc
UILabel *myLable = [[UILabel alloc] initWithFrame:CGRectMake(0, 0, 320, 21)];
[myLable setText:@"My Test Label"];
[self.view addSubview:myLable];
```
<img src="/images/article/custom_navigation_bar/iOS7_navigation_bar_status_bar.png"/>

<img src="/images/article/custom_navigation_bar/iOS6_navigation_bar_status_bar.png"/>

Tips:

> 所以在进行开发时我们要从充分虑到视图的Layout,建议你的ViewController都继承自基类BaseViewController,并在其中做layout处理.

你可以用如下方法判断iOS版本:

``` objc
#define OS_PRIOR_IOS_7 (floor(NSFoundationVersionNumber) <= NSFoundationVersionNumber_iOS_6_1)

if (OS_PRIOR_IOS_7) {
    // Load resources for iOS 6.1 or earlier
} else {
    // Load resources for iOS 7 or later
}
```

##[定制返回按钮的颜色](id:anchor5)

iOS7:

导航栏上的按钮都是无边框的。要想给返回按钮着色，可以使用tintColor属性。如下代码所示：
``` objc
    [[UINavigationBar appearance] setTintColor:[UIColor whiteColor]];
```
如果想要用自己的图片替换返回图标，可以设置图片的`backIndicatorImage`和`backIndicatorTransitionMaskImage`。如下代码所示：
``` objc
    [[UINavigationBar appearance] setBackIndicatorImage:[UIImage imageNamed:@"back_btn.png"]];
    [[UINavigationBar appearance] setBackIndicatorTransitionMaskImage:[UIImage imageNamed:@"back_btn.png"]];
```

<img src="/images/article/custom_navigation_bar/IOS7_navigation_bar_cutom_back_btn.png"/>

iOS6:

导航栏上的按钮都是有边框的。要想改变返回按钮UI, 可以使用以下方法：
``` objc
NSDictionary *barItemAttributes = [NSDictionary dictionaryWithObjectsAndKeys:
                               UIColorFromRGB(0xffffff), UITextAttributeTextColor,
                               UIColorFromRGB(0x000000), UITextAttributeTextShadowColor,
                               [NSValue valueWithUIOffset:UIOffsetMake(0, 1)], UITextAttributeTextShadowOffset,
                               [UIFont fontWithName:@"AmericanTypewriter" size:0.0], UITextAttributeFont,
                               nil];

[[UIBarButtonItem appearance] setTitleTextAttributes:barItemAttributes forState:UIControlStateNormal];

UIImage *buttonBack30 = [[UIImage imageNamed:@"button_back_textured_30"] resizableImageWithCapInsets:UIEdgeInsetsMake(0, 13, 0, 5)];
UIImage *buttonBack24 = [[UIImage imageNamed:@"button_back_textured_24"] resizableImageWithCapInsets:UIEdgeInsetsMake(0, 12, 0, 5)];
[[UIBarButtonItem appearance] setBackButtonBackgroundImage:buttonBack30 forState:UIControlStateNormal barMetrics:UIBarMetricsDefault];
[[UIBarButtonItem appearance] setBackButtonBackgroundImage:buttonBack24 forState:UIControlStateNormal barMetrics:UIBarMetricsLandscapePhone];
```
<img src="/images/article/custom_navigation_bar/IOS6_navigation_bar_cutom_back_btn.png"/>

为了更好的兼容横屏模式你需要准备两种尺寸的背景图: 

<img src="/images/article/custom_navigation_bar/button_back_textured_24.png"/>
48x24

<img src="/images/article/custom_navigation_bar/button_back_textured_24.png"/>
48x30


##[修改导航栏标题的字体](id:anchor5)

跟iOS 6一样，我们可以使用导航栏的titleTextAttributes属性来定制导航栏的文字风格。<br>在text attributes字典中使用如下一些key，可以指定字体、文字颜色、文字阴影色以及文字阴影偏移量：

iOS5/6:
``` objc
UIFont *titleFont = [UIFont fontWithName:@"HelveticaNeue-CondensedBlack" size:21.0];
NSDictionary *titleTextAttributes = @{UITextAttributeTextColor : UIColorFromRGB(0xf5f5f5),
                                      UITextAttributeTextShadowColor : UIColorFromRGB(0x000000),
                                      UITextAttributeTextShadowOffset : [NSValue valueWithUIOffset:UIOffsetMake(0, 1)],
                                      UITextAttributeFont : titleFont};
[[UINavigationBar appearance] setTitleTextAttributes:titleTextAttributes];
```

iOS7:
``` objc
NSShadow *shadow = [[NSShadow alloc] init];
shadow.shadowColor = [UIColor colorWithRed:0.0 green:0.0 blue:0.0 alpha:0.8];
shadow.shadowOffset = CGSizeMake(0, 1);

UIFont *titleFont = [UIFont fontWithName:@"HelveticaNeue-CondensedBlack" size:21.0];
NSDictionary *titleTextAttributes = @{NSForegroundColorAttributeName: UIColorFromRGB(0xf5f5f5),
                                      NSShadowAttributeName : shadow,
                                      NSFontAttributeName : titleFont};

[[UINavigationBar appearance] setTitleTextAttributes:titleTextAttributes];
```

<img src="/images/article/custom_navigation_bar/contrast_navigation_bar_custom_title.png"/>

##[修改导航栏标题为图片](id:anchor7)

如果要想将导航栏标题修改为一个图片或者logo，那么只需要使用下面这行代码即可:
``` objc
self.navigationItem.titleView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"app_title"]];
```
上面的代码简单的修改了titleView属性，将一个图片赋值给它。 注意：这不是iOS 7中的新功能，之前的iOS版本就可以已经有了。

具体效果如下图所示：

<img src="/images/article/custom_navigation_bar/contrast_navigation_bar_custom_image_title.png"/>

##[添加多个按钮](id:anchor8)

同样,这个技巧也不是iOS 7的,开发者经常会在导航栏中添加多个按钮,所以我决定在这里进行介绍.

我们可以在导航栏左边或者右边添加多个按钮.例如,我们希望在导航栏右边添加一个照相机和分享按钮,那只需要使用下面的代码即可:
``` objc
UIBarButtonItem *shareItem = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemAction target:self action:nil];
UIBarButtonItem *cameraItem = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemCamera target:self action:nil];

NSArray *actionButtonItems = @[shareItem, cameraItem];
self.navigationItem.rightBarButtonItems = actionButtonItems;
```
在iOS6上系统提供样式不够好? 当然你可以用自己的图片来完成想要的效果:
``` objc
NSDictionary *barItemAttributes = [NSDictionary dictionaryWithObjectsAndKeys:
                                   UIColorFromRGB(0xffffff), UITextAttributeTextColor,
                                   UIColorFromRGB(0x000000), UITextAttributeTextShadowColor,
                                   [NSValue valueWithUIOffset:UIOffsetMake(0, 1)], UITextAttributeTextShadowOffset,
                                   [UIFont fontWithName:@"AmericanTypewriter" size:0.0], UITextAttributeFont,
                                   nil];

[[UIBarButtonItem appearance] setTitleTextAttributes:barItemAttributes forState:UIControlStateNormal];

UIImage *button30 = [[UIImage imageNamed:@"button_textured_30"] resizableImageWithCapInsets:UIEdgeInsetsMake(0, 13, 0, 5)];
UIImage *button24 = [[UIImage imageNamed:@"button_textured_24"] resizableImageWithCapInsets:UIEdgeInsetsMake(0, 12, 0, 5)];
[[UIBarButtonItem appearance] setBackgroundImage:button30 forState:UIControlStateNormal barMetrics:UIBarMetricsDefault];
[[UIBarButtonItem appearance] setBackButtonBackgroundImage:button24 forState:UIControlStateNormal barMetrics:UIBarMetricsLandscapePhone];        
```

<img src="/images/article/custom_navigation_bar/contrast_navigation_bar_muti_items.png"/>


##[修改状态栏的风格](id:anchor9)

在老版本的iOS中,状态栏永远都是白色风格.而在iOS 7中,我们可以修改每个view controller中状态栏的外观.

我们需要在对应的view controller中复写 `preferredStatusBarStyle:`方法
``` objc
- (UIStatusBarStyle)preferredStatusBarStyle
{
  return UIStatusBarStyleLightContent;
}
```
需要注意的是如果你的ViewController套在NavigationController中你只能通过改NavigationController的`preferredStatusBarStyle`才能起到效果.

如果你用的是xib或者storyboard那你可以在xib中设置对应视图的status bar style.

<img src="/images/article/custom_navigation_bar/xib_status_bar.png"/>

还有另外一种方法来改变status bar的样式(`推荐`):

可以使用UIApplication的statusBarStyle方法来设置状态栏,不过,首先需要停止使用`View controller-based status bar appearance`.

在`project target`的`Info tab`中,插入一个新的key,名字为`View controller-based status bar appearance`,并将其值设置为NO.

<img src="/images/article/custom_navigation_bar/modify_info_status_bar_style.png"/>

之后你就可以通过下面的代码来改变status bar 的风格了
``` objc
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent];
```

##[隐藏状态栏](id:anchor10)

有时候我们需要隐藏状态栏,那么此时我们在view controller中override方法prefersStatusBarHidden:即可,如下代码所示:

iOS6:

``` objc
[[UIApplication sharedApplication] setStatusBarHidden:YES];
```
iOS7:

``` objc
- (BOOL)prefersStatusBarHidden
{
  return YES;
}
```

##[接下来?](id:anchor11)

还会继续总结Custom UI相关文章

你可以在这里下载到本文的[Demo](https://github.com/goldfish0506/iOSCustomUI/archive/master.zip),(本文提供的代码需要用Xcode 5来执行)

如果文中有错误的地方,请指正,谢谢~
