---
layout: post
title:  "Remove a Color from UIImage"
date:   2016-02-16 0:00:55
categories: iOS
comments: true
---

刪除顏色

<script src="https://gist.github.com/fengyitsai/5be0160c7617734f6585.js"></script>


{% highlight objc %}
//Ref:http://stackoverflow.com/questions/633722/how-to-make-one-color-transparent-on-a-uiimage
- (UIImage*) removeColor:(UIColor*)color inImage:(UIImage*)image withTolerance:(float)tolerance {
  
  CGImageRef imageRef = [image CGImage];
  NSUInteger width = CGImageGetWidth(imageRef);
  NSUInteger height = CGImageGetHeight(imageRef);
  CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
  
  NSUInteger bytesPerPixel = 4;
  NSUInteger bytesPerRow = bytesPerPixel * width;
  NSUInteger bitsPerComponent = 8;
  NSUInteger bitmapByteCount = bytesPerRow * height;

  CGContextRef context = CGBitmapContextCreate(nil, width, height,
                                               bitsPerComponent, bytesPerRow, colorSpace,
                                               kCGImageAlphaPremultipliedLast | kCGBitmapByteOrder32Big);
  
  
  unsigned char *rawData = CGBitmapContextGetData(context);
  
  CGColorSpaceRelease(colorSpace);
  
  CGContextDrawImage(context, CGRectMake(0, 0, width, height), imageRef);
  
  size_t numComponents = CGColorGetNumberOfComponents(color.CGColor);
  
  float r = 0;
  float g = 0;
  float b = 0;
  
  if(numComponents == 2){
    const CGFloat* colors = CGColorGetComponents( color.CGColor );
    r = colors[0] * 255.0;
    g = colors[0] * 255.0;
    b = colors[0] * 255.0;
  }else if(numComponents == 4){
    const CGFloat* colors = CGColorGetComponents( color.CGColor );
    r = colors[0] * 255.0;
    g = colors[1] * 255.0;
    b = colors[2] * 255.0;
  }
  
  const float redRange[2] = {
    MAX(r - tolerance, 0.0),
    MIN(r + tolerance, 255.0)
  };
  
  const float greenRange[2] = {
    MAX(g - tolerance, 0.0),
    MIN(g + tolerance, 255.0)
  };
  
  const float blueRange[2] = {
    MAX(b - tolerance, 0.0),
    MIN(b + tolerance, 255.0)
  };
  
  int byteIndex = 0;
  
  while (byteIndex < bitmapByteCount) {
    float red   = rawData[byteIndex];
    float green = rawData[byteIndex + 1];
    float blue  = rawData[byteIndex + 2];
    
    if (((red >= redRange[0]) && (red <= redRange[1])) &&
        ((green >= greenRange[0]) && (green <= greenRange[1])) &&
        ((blue >= blueRange[0]) && (blue <= blueRange[1]))) {

      rawData[byteIndex + 0] = 0;
      rawData[byteIndex + 1] = 0;
      rawData[byteIndex + 2] = 0;
      rawData[byteIndex + 3] = 0;
    }
    
    byteIndex += 4;
  }
  
  CGImageRef imgref = CGBitmapContextCreateImage(context);
  UIImage *result = [UIImage imageWithCGImage:imgref];
  
  CGImageRelease(imgref);
  CGContextRelease(context);

  return result;
}
{% endhighlight %}