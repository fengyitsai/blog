---
layout: post
title:  "有效率的把UIView轉成指定大小的圖片"
date:   2016-06-03 0:05:55
categories: ios
comments: true
---
###  有效率的把UIView轉成指定大小的圖片

一般看的的做法是透過 CGLayer的renderInContext 轉成UIImage,  
再[UIImage drawInRect:...] 畫成想要的大小

但這樣有點浪費效能  
可以直接在renderInContext階段完成所有操作  

```obj-c  

+(UIImage *)imageWithView:(UIView *)view inSize:(CGSize) size
{
  UIGraphicsBeginImageContextWithOptions(size, NO, 0.0);
  CGContextRef context = UIGraphicsGetCurrentContext();
  
  CGContextScaleCTM(context, size.width/view.bounds.size.width, size.height/view.bounds.size.height);
  [[view layer] renderInContext:UIGraphicsGetCurrentContext()];
  
  UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
  UIGraphicsEndImageContext();
  return newImage;
}

```  
