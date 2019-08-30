---
title : "对算法笔试题的总结"
date: 2019-08-29T01:37:56+08:00
draft: false
tags: ["Java", "Algorithm"]
categories: ["Algorithm"]
author: "Yang Zhang"
---

算法笔试目前看来更多的是通过题目抽象出具体算法要求, 然后根据经验写出计算过程. 因此, 总结一些经典的方法和做过的题目还是很有必要的. 这篇博客用来拾遗用, 把做过的东西都捋一捋.

### String Related
#### 子串问题
子串问题最常见的有求最长子串, 最短子串等问题, 与sequence不同, 子串的特点是连续性, 所以最优解一定是遍历一次字符串后就可以得到答案, 复杂度应为O(n). 例如76. Minimum Window Substring(string s 中包含string t的最小子串)
```c++
string minWindow(string s, string t) {
        vector<int> map(128,0);
        for(auto c: t) map[c]++;
        int counter=t.size(), begin=0, end=0, d=INT_MAX, head=0;
        while(end<s.size()){
            if(map[s[end++]]-->0) counter--; //in t
            while(counter==0){ //valid
                if(end-begin<d)  d=end-(head=begin);
                if(map[s[begin++]]++==0) counter++;  //make it invalid
            }  
        }
        return d==INT_MAX? "":s.substr(head, d);
    }
```
思路为构建一个128长度的数组(ASCII)来记录出现的次数. 首先把target的频率加入到数组中. 然后把遍历source, 直到counter为0时,证明当前子串包括了target所有的字符. 然后开始修剪部分.

从头开始判断头部的字符可不可以丢掉. 当前数组的状态应该为: target中有的字符, 对应位置数值是0. source中有但target中没有的字符, 对应数值是负数. 两方都没有的字符, 对应数值应该是0. 所以要做的事情有两个:

1. 尝试更新最小字符串的长度.
2. 去除首个字符, 使其刚好不能构成包含所有target字符的子串.

第二步理解了挺久的. 总的来说就是想要找到新的满足条件的子串必须在后面的字符中先找到第一个在target里的字符, 然后删除掉之前的字符,就是构成新的字符串. 这里代码编写的反了过来, 先把第一个字符删除, 然后看有没有新的可以替代他.

最终我们可以得到想要的子串. 通过这个例子我们可以找到解答子串问题的通用模版
```Java
int findSubstring(string s){
        vector<int> map(128,0);
        int counter; // check whether the substring is valid
        int begin=0, end=0; //two pointers, one point to tail and one  head
        int d; //the length of substring

        for() { /* initialize the hash map here */ }

        while(end<s.size()){

            if(map[s[end++]]-- ?){  /* modify counter here */ }

            while(/* counter condition */){ 
                 
                 /* update d here if finding minimum*/

                //increase begin to make it invalid/valid again
                
                if(map[s[begin++]]++ ?){ /*modify counter here*/ }
            }  

            /* update d here if finding maximum*/
        }
        return d;
  }
```