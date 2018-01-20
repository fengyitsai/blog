---
layout: post
title:  "LeetCode: Median of Two Sorted Arrays"
date:   2016-05-23 0:00:55
categories: leetcode
comments: true
---
###  LeetCode: Median of Two Sorted Arrays

There are two sorted arrays nums1 and nums2 of size m and n respectively. Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

關鍵點在於要如何有效率地忽略array中不必要的item.  
假設我們要找第N個item,  
這邊使用了index1, index2 分別標示出nums1,nums2 中的兩個位置  
而其中 index1+index2 要等於 N-2
這時會有三種情況:  

**1.  nums1[index1] > nums2[index2]**  
	*	表示 nums2[index2] (包含) 之前的item都不可能是第N個, 因此可以忽略, 並把N-(index2+1)   (註1)  
**2.  nums1[index1] < nums2[index2]**  
 	*	表示 nums1[index1] (包含) 之前的item都不可能是第N個, 因此可以忽略, 並把N-(index1+1)  
**3.  nums1[index1] = nums2[index2]**  
	*   表示 nums1[index1] nums2[index2] 都可以是第N個  
遞迴可得出最後答案.

> 註1:  
> 假設nums2[index2] 是第N個數, 那前面必須要有N-1個比較小的item,   
> 而 nums1[index1] 比 nums2[index2] 大, 所以前面最多有 index1 個item比 nums2[index2] 小(<=)   
> 而index2 前面也有 index2個item比 nums2[index2] 小(<=)
> 所以加起來是 index1 + index2,  
> 因為 index1+index2 等於 N-2,  
> 由 N-2 < N-1 可知 nums2[index2] 不可能第N個數



```obj-c  
class Solution {
  func findMedianSortedArrays(nums1: [Int], _ nums2: [Int]) -> Double {
    
    func findNthInSortedArrays(k:Int, _ nums1: [Int], _ nums2: [Int]) -> Int{

      if(nums1.count == 0){
        return nums2[k-1];
      }
      
      if(nums2.count == 0){
        return nums1[k-1];
      }
      
      if(k <= 1){
        if(nums1[0] < nums2[0]){
          return nums1[0];
        }else{
          return nums2[0];
        }
      }
      
      var index1:Int = nums1.count/2;
      var index2:Int = k-index1-2;
      
      if(index2 < 0){
        index1 = 0;
        index2 = k-index1-2;
      }
      
      if(index2 >= nums2.count){
        index1 = nums1.count-1;
        index2 = k-index1-2;
        
        if(index2 < 0){
          index2 = 0;
          index1 = k-index2-2;
        }
      }

      if(nums1[index1] > nums2[index2]){
        return findNthInSortedArrays(k-index2-1,nums1,Array(nums2.suffixFrom(index2+1)));
      }
      
      if(nums1[index1] < nums2[index2]){
        return findNthInSortedArrays(k-index1-1,Array(nums1.suffixFrom(index1+1)),nums2);
      }
      
      return nums1[index1];
    }
    
    let totalCount = nums1.count + nums2.count;
    
    if((totalCount)%2 == 0){
      
      let first = Double(findNthInSortedArrays(totalCount/2,nums1, nums2));
      let second = Double(findNthInSortedArrays(totalCount/2+1,nums1, nums2));
      
      //print("first:\(first)  second:\(second)")
      
      
      return (first+second)/2 ;
    }else{
      let k = totalCount/2 + 1;
      
      return Double(findNthInSortedArrays(k,nums1, nums2));
    }
  }
}

```  
