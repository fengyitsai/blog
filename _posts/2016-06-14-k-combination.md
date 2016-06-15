---
layout: post
title:  "swift k-combination"
date:   2016-06-14 0:05:55
categories: swift
comments: true
---
###  k-combination

```obj-c

func combinations<T>(array:[T], size:Int) -> [[T]] {
  if(size > array.count || size == 0 || array.count == 0){
    return [];
  }
  
  if(size == array.count){
    return [array];
  }
  
  var combs:[[T]] = [];
  
  if(size == 1){
    for item in array {
      combs.append([item]);
    }
    
    return combs;
  }
  
  for i in 0...(array.count-size) {
    let tails = combinations(Array(array[(i+1)..<array.endIndex]), size: size-1)
    for tail in tails {
      let element = [array[i]] + tail;
      combs.append(element);
    }
  }
  
  return combs;
}
```