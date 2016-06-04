---
layout: post
title:  "LeetCode: ZigZag Conversion"
date:   2016-06-04 0:05:55
categories: leetcode
comments: true
---
###  LeetCode: ZigZag Conversion

```obj-c

class Solution {
  func convert(s: String, _ numRows: Int) -> String {
    if(numRows <= 1){
      return s;
    }
    
    let characters = Array(s.characters);
    var numBlocks = (characters.count/(2*numRows-2))+1;
    let numBlockItems = (2*numRows-2);

    var out = "";

    for rowIndex in 0..<numRows {
      for blockIndex in 0..<numBlocks {
        
        let i1 = rowIndex+blockIndex*(numBlockItems);
        
        if(i1>=characters.count){
          continue;
        }
        
        if(rowIndex == 0){
          out = out + String(characters[i1]);
          continue;
        }
        
        if(rowIndex == numRows-1){
          out = out + String(characters[i1]);
          continue;
        }
        
        let i2 = (numBlockItems-rowIndex)+blockIndex*(numBlockItems);
        
        out = out + String(characters[i1]);
        
        if(i2<characters.count){
          out = out + String(characters[i2]);
        }
      }
    }

    return out;
  }
}
```