---
layout: post
title:  "Method Swizzle"
date:   2016-04-26 0:00:55
categories: ios
comments: true
---
Method Swizzle 顧名思義是用來交換兩個Method,  
實務上常常用來hook某個api, 因為實現容易其耦合程度非常低.  
例如想統計所有UIViewController 的viewWillAppear,viewWillDisappear,  
原本可能需要修改每個UIViewController,  或增加父類來實現.  
但使用Method Swizzle 只需要新增一個Category就可完成這項任務, 而原全部污染其他代碼.  

使用 Method Swizzle 基本上只需要三件事  
1. 拿到各自的SEL   
2. 用各自的SEL拿到各自的Method  
3. method_exchangeImplementations(method1,method2)  

代碼寫出來會像下面這樣:  

```obj-c
@implementation ClassB(Tracking)
+ (void)load {
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    Class class = [self class];
    
    SEL functionSEL = @selector(function);
    SEL xxx_functionSEL = @selector(xxx_function);
    
    Method functionMethod = class_getInstanceMethod(class, functionSEL);
    Method xxx_functionMethod = class_getInstanceMethod(class, xxx_functionSEL);
    
    method_exchangeImplementations(functionMethod, xxx_functionMethod);
  });
}

- (void)xxx_function{
  [self xxx_function];
}
@end
```

>其中為什麼要使用+load, dispatch_once, [self xxx_function] 請參考Mattt的[這篇文章](http://nshipster.com/method-swizzling/)  
>**[self xxx_function] 這行非常重要, 如果拿掉等於取消原本function的行為**  

上述的代碼在一般的情況是沒有問題的,
但如果ClassB的"function"是從superclass來(假設為ClassA), 就會產生嚴重的問題,

原本使用ClassA調用的function, 會被轉到ClassB的xxx_function, 而此時執行的 [self xxx_function],  
會產生NSInvalidArgumentException, 因為 ClassA 中並沒有 xxx_function 這個selector.  
  
**圖示**  

> **原本為:**
> ClassA:
> functionSEL -> functionMethod
> 
> ClassB:
> xxx_functionSEL -> xxx_functionMethod
> 
> **method_exchangeImplementations後:**
> ClassA:
> functionSEL -> xxx_functionMethod
> 
> ClassB:
> xxx_functionSEL -> functionMethod
  
上面可看出, 原本我們只是要改變ClassB的行為, 但無意中影響到了ClassA, 明顯表示這樣的實作是有問題的.


**所以在Mattt的文中提供了以下這種做法:**(為了保持一致性, 這邊修改了class和method的名稱)  
-------------

```obj-c

@implementation ClassB (Tracking)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];

	    SEL functionSEL = @selector(function);
	    SEL xxx_functionSEL = @selector(xxx_function);
    
	    Method functionMethod = class_getInstanceMethod(class, functionSEL);
	    Method xxx_functionMethod = class_getInstanceMethod(class, xxx_functionSEL);

        BOOL didAddMethod =
            class_addMethod(class,
                selector1,
                method_getImplementation(xxx_functionMethod),
                method_getTypeEncoding(xxx_functionMethod));

        if (didAddMethod) {
            class_replaceMethod(class,
                selector2,
                method_getImplementation(functionMethod),
                method_getTypeEncoding(functionMethod));
        } else {
            method_exchangeImplementations(functionMethod, xxx_functionMethod);
        }
    });
}

- (void)xxx_function{
  [self xxx_function];
}
@end
```
  
  
這邊使用了class_addMethod為ClassB 增加selector為 “functionSEL”, Method為 "xxx_functionMethod". 的method.  

> functionSEL -> xxx_functionMethod

如果添加成功表示selector1來自ClassA.  
之後就直接把原本的xxx_function 指向 functionMethod  

> 原本為:
> ClassB:
> functionSEL -> xxx_functionMethod
> xxx_functionSEL -> xxx_functionMethod
> 
> replaceMethod之後:
> ClassB:
> functionSEL -> xxx_functionMethod
> xxx_functionSEL -> functionMethod

這樣就不會影響到ClassA了  



### 參考: 
<http://nshipster.com/method-swizzling/>
<https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/tag/objc_method_description>


```obj-c
typedef struct objc_method *Method;

struct objc_method {
    SEL method_name;
    char *method_types;
    IMP method_imp;
}
```
