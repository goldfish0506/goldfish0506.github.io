---
layout: post
title: "NSAttributedString 计算高度的天坑..."
date: 2014-05-08 17:03:21 +0800
comments: true
categories: iOS, NSAttributedString
---

苹果对富文本的支持越来越完备, 在iOS7中更好地支持HTML了,现在你可以用NSAttributedString来展示HTML的内容.

尝试使用`NSHTMLTextDocumentType`展示HTML吧:

{% codeblock lang:objc %}

NSString *html = @"<bold>Wow!</bold> Now <em>iOS</em> can create <h3>NSAttributedString</h3> from HTMLs!";
NSDictionary *options = @{NSDocumentTypeDocumentAttribute: NSHTMLTextDocumentType};

NSAttributedString *attrString = [[NSAttributedString alloc] initWithData:[html dataUsingEncoding:NSUTF8StringEncoding]
                                                                  options:options
                                                       documentAttributes:nil
                                                                    error:nil];

{% endcodeblock %}

同样你也可以通过NSAttributedString获得HTML文本:

{% codeblock lang:objc %}

NSAttributedString *attrString; // from previous code
NSDictionary *options = @{NSDocumentTypeDocumentAttribute: NSHTMLTextDocumentType};

NSData *htmlData = [attrString dataFromRange:NSMakeRange(0, [attrString length]) documentAttributes:options error:nil];
NSString *htmlString = [[NSString alloc] initWithData:htmlData encoding:NSUTF8StringEncoding];

{% endcodeblock %}

但是再享受NSAttributedString带来方便的同时,我们也会遇到些令人头痛的小麻烦...

<!-- more -->

比如在计算高度的时候我就遇到了个蛋疼的问题:

这里介绍下计算高度的方法...需要的同学拿去吧~
或者你也可以从Mattt大神哪里获得[一个更牛逼的计算方法](https://github.com/mattt/TTTAttributedLabel/blob/master/TTTAttributedLabel/TTTAttributedLabel.m#L393-L395)

{% codeblock lang:objc %}

@interface NSString (Calculate)

- (CGSize)sizeWithAttributes:(NSDictionary *)attrs constrainedToSize:(CGSize)size;

@end

@implementation NSString (Calculate)

- (CGSize)sizeWithAttributes:(NSDictionary *)attrs constrainedToSize:(CGSize)size
{
  NSAttributedString *attributedString = [[NSAttributedString alloc] initWithString:self
                                                                         attributes:attrs];
  CTFramesetterRef framesetter = CTFramesetterCreateWithAttributedString((__bridge CFAttributedStringRef) attributedString);
  CGSize fitSize = CTFramesetterSuggestFrameSizeWithConstraints(framesetter, CFRangeMake(0, 0), NULL, size, NULL);
  CFRelease(framesetter);
  return fitSize;
}

@end

{% endcodeblock %}

故事到这里看似都顺风顺水

<img src="http://alvin-blog.qiniudn.com/u=3540161728,54641166&fm=56.jpeg" />

{% codeblock lang:objc %}
  NSMutableParagraphStyle *paragraphStyle = [[NSMutableParagraphStyle alloc] init];
  paragraphStyle.lineSpacing = 3;
  paragraphStyle.lineBreakMode = NSLineBreakByTruncatingTail;

  NSDictionary *attributes = @{NSFontAttributeName : [UIFont systemFontOfSize:12],
                               NSParagraphStyleAttributeName : paragraphStyle}
{% endcodeblock %}

直到我将NSMutableParagraphStyle的`lineBreakMode`设置成`NSLineBreakByTruncatingTail`之后,就开始蛋疼了...
这个时候返回的高度永远都只是单行的高度...

<img src="http://alvin-blog.qiniudn.com/%5DOI%5BT%5BQ45LY_U@JT)OVW%6059.jpg" />

百般寻觅之后终于知道了问题所在:

> lineBreakMode 设置成 NSLineBreakByTruncatingTail | NSLineBreakByTruncatingHead | NSLineBreakByTruncatingMiddle 在计算高度的时候 会被系统默认成单行

所以这里需要为计算高度的时候单独设置一个paragraphStyle,不用担心,在这里计算得到高度与最后显示的结果是一致的...

{% codeblock lang:objc %}

+ (NSDictionary *)attributesForUserTextWithCalculateMode:(BOOL)isCalculateMode
{
  NSMutableParagraphStyle *paragraphStyle = [[NSMutableParagraphStyle alloc] init];
  paragraphStyle.lineSpacing = 3;
  paragraphStyle.lineBreakMode = isCalculateMode ? NSLineBreakByWordWrapping : NSLineBreakByTruncatingTail;

  return @{NSFontAttributeName : [UIFont systemFontOfSize:12],
           NSParagraphStyleAttributeName : paragraphStyle,
           NSForegroundColorAttributeName: [UIColor blackColor]};
}

{% endcodeblock %}

木哈哈...这样看起来好多了≖‿≖✧

<img width="400" src="http://alvin-blog.qiniudn.com/48B36488-3D3E-45F2-A5C1-1EB55BE81EAA.png" />


参考链接:

http://lists.apple.com/archives/cocoa-dev/2012/Nov/msg00179.html
http://stackoverflow.com/questions/7378308/uibuttons-title-label-word-wrap-with-tail-truncation



