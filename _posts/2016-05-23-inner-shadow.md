---
layout: post
title:  "Inner Shadow"
date:   2016-05-23 0:00:55
categories: ios
comments: true
---

## Inner Shadow

CoreGraphics只提供了一般陰影的API, 要做到內陰影需要自行處理  
而產生內陰影需要3到4個步驟:

原始圖
![origin](http://fengyitsai.github.io/blog/assets/images/innerShadow/origin.png)
1. 先產生黑白相反的圖(以下稱mask)  
![mask](http://fengyitsai.github.io/blog/assets/images/innerShadow/mask.png)
2. 以原圖當作遮罩 繪製mask的陰影  
![shadow](http://fengyitsai.github.io/blog/assets/images/innerShadow/shadow.png)  
3. 以mask當做遮罩 繪製光線(optional)
![mask](http://fengyitsai.github.io/blog/assets/images/innerShadow/light.png)
4. 合成步驟2.3.產生的圖
![merge](http://fengyitsai.github.io/blog/assets/images/innerShadow/merge.png)

  
```obj-c
//步驟一: 先產生黑白相反的圖(以下稱mask)  
+ (UIImage *)inverseImage:(UIImage *)image{
    CGRect rect = { CGPointZero, image.size };
    CGFloat scale = image.scale;
    UIGraphicsBeginImageContextWithOptions(rect.size, NO, scale);
    
    CGContextRef context = UIGraphicsGetCurrentContext();

    [[UIColor blackColor] setFill];
    UIRectFill(rect);
    
    CGContextTranslateCTM(context, 0, image.size.height);
    CGContextScaleCTM(context, 1.0, -1.0);
    
    CGContextClipToMask(UIGraphicsGetCurrentContext(), rect, image.CGImage);
    CGContextClearRect(UIGraphicsGetCurrentContext(), rect);

    
    UIImage *output = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return output;
}
```

```obj-c
//步驟二: 以原圖當作遮罩 繪製mask的陰影
+ (UIImage *)shadowFromImage:(UIImage *)image mask:(UIImage *)mask texture:(UIColor *)texture shadowColor:(UIColor *) shadowColor shadowBlur:(CGFloat)shadowBlur shadowOffset:(CGSize) shadowOffset{
    CGRect rect = { CGPointZero, mask.size };
    UIGraphicsBeginImageContextWithOptions(rect.size, NO, mask.scale);
    CGContextRef context = UIGraphicsGetCurrentContext();

    CGContextTranslateCTM(context, 0, mask.size.height);
    CGContextScaleCTM(context, 1.0, -1.0);

    CGContextClipToMask(context, rect, mask.CGImage);

    [texture setFill];

    CGContextFillRect(context, rect);

    CGContextScaleCTM(context, 1.0, -1.0);
    CGContextTranslateCTM(context, 0, -mask.size.height);
    CGContextSetShadowWithColor(context, shadowOffset, shadowBlur, shadowColor.CGColor);
    [image drawAtPoint:CGPointZero];

    UIImage *output = UIGraphicsGetImageFromCurrentImageContext();

    UIGraphicsEndImageContext();
    return output;
}
```

```obj-c
//步驟三: 以mask當做遮罩 繪製光線(optional)
+ (UIImage *)lightOnImage:(UIImage *)image mask:(UIImage *)mask lightColor:(UIColor *) lightColor lightBlur:(CGFloat)lightBlur lightOffset:(CGSize) lightOffset{
    CGRect rect = { CGPointZero, image.size };

    UIGraphicsBeginImageContextWithOptions(rect.size, NO, image.scale);

    CGContextRef context = UIGraphicsGetCurrentContext();
    
    CGContextTranslateCTM(context, 0, image.size.height);
    CGContextScaleCTM(context, 1.0, -1.0);
    CGContextClipToMask(context, rect, mask.CGImage);

    CGContextScaleCTM(context, 1.0, -1.0);
    CGContextTranslateCTM(context, 0, -image.size.height);
    CGContextSetShadowWithColor(context, lightOffset, lightBlur, lightColor.CGColor);
    [image drawAtPoint:CGPointZero];
    
    UIImage *output = UIGraphicsGetImageFromCurrentImageContext();

    UIGraphicsEndImageContext();
    return output;
}

```


```obj-c
//步驟四: 合成步驟2.3.產生的圖
+(UIImage*)createCombineImage:(NSArray*)images{
    UIImage* firstImg = images.firstObject;
    
    CGRect rect = { CGPointZero, firstImg.size };
    
    UIGraphicsBeginImageContextWithOptions(rect.size, NO, firstImg.scale);
    CGContextRef ctxt = UIGraphicsGetCurrentContext();
    
    CGContextTranslateCTM(ctxt, 0.f, rect.size.height);
    CGContextScaleCTM(ctxt, 1.f, -1.f);
    
    for (UIImage* img in images){
      CGContextDrawImage(ctxt, rect, img.CGImage);
    }
    
    UIImage* output = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return output;
}
```