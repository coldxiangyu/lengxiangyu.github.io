---
layout: post
title:  "从头学算法（三、LeetCode之 Two Sum）"
date:   2016-06-25 10:13:22
categories: 数据结构及算法
tags: algorithm
mathjax: true
---

刚刚注册完LeetCode，打算找个软柿子捏一下，然后在Algorithms目录下看到排在第一位的是一个Two Sum的例子。那就拿它开刀吧。
首先看看这个题是怎样的：
![image_1bf96poeifoa22p1vcl1v5u6el9.png-32.2kB][1]




简单来说，就是给你一个数组，一个目标值，你要在这个数组中找到任意两个值相加的和是所给目标值，并返回这两个值在数组中的下标。
这个简直简单啊，一个双循环就搞定了。
以下是我提交的代码，一次性编译成功，AC：
``` java
public class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] back = new int[2];
        for(int i=0;i<nums.length-1;i++){
            for(int j=nums.length-1;j>i;j--){
                if(nums[i]+nums[j]==target){
                    back[0]=i;
                    back[1]=j;
                }
            }
        }
        return back;
    }
}
```
开心还没两秒，我发现一个问题：
![image_1bf95cc6e155n17u1ggedr34vdp.png-26.6kB][2]
LeetCode还提供了Accepted Solutions Runtime Distribution，我看了下自己的位置，Your runtime beats 5.97% of java submissions!!!!才5.97%是什么鬼。
我才开始意识到自己之前的编程习惯是错误的，不能一个功能实现就好了，还要考虑算法是不是最优的，要考虑时间复杂度。
我突然对算法又增加了几分好感，有挑战才是最好的嘛。
so，我开始考虑如何进行优化：
我的双层循环是一正一反的，也就是第一层正向，第二层逆向，我们知道，冒泡排序的时候正向冒泡和逆向冒泡是有区别的，但整体时间复杂度没有区别，但对于这个问题就不一样了，正向明显要好于逆向。
因此优化如下：
``` java
public class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] back = new int[2];
        for(int i=0;i<nums.length;i++){
            for(int j=i+1;j<nums.length;j++){
                if(nums[i]+nums[j]==target){
                    back[0]=i;
                    back[1]=j;
                    break;
                }
            }
        }
        return back;
    }
}
```
效率图：
![image_1bf996mcs1448189s1lsa1j0k1aih9.png-27.3kB][3]
从5.97%直奔16.29%你敢信！

然后我又参考了一下其他靠前的代码：
``` java
public class Solution {
    public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
            for (int j = 0; j < nums.length; j++) {
                if(nums[i] + nums[j] == target &&
                        i != j) {
                    return new int[] {i, j};
                }
            }
        }
        return new int[] {};
    }
}
```
这个跟我的没什么区别啊，完全不懂是怎么排到前面的，也不晓得LeetCode具体依据什么进行的runtime排序，感觉不是很准。
另外贴一下vote率最高的java实现，拓展下思路，利用map的containsKey方法：
```java
public int[] twoSum(int[] numbers, int target) {
    int[] result = new int[2];
    Map<Integer, Integer> map = new HashMap<Integer, Integer>();
    for (int i = 0; i < numbers.length; i++) {
        if (map.containsKey(target - numbers[i])) {
            result[1] = i + 1;
            result[0] = map.get(target - numbers[i]);
            return result;
        }
        map.put(numbers[i], i + 1);
    }
    return result;
}
```

  [1]: http://static.zybuluo.com/coldxiangyu/w3ho0r5zegrtwds2df4qh5a5/image_1bf96poeifoa22p1vcl1v5u6el9.png
  [2]: http://static.zybuluo.com/coldxiangyu/1jwgx9mnanm1ppfglu72fd1i/image_1bf95cc6e155n17u1ggedr34vdp.png
  [3]: http://static.zybuluo.com/coldxiangyu/xzz1eu01kyo88d6youxezue1/image_1bf996mcs1448189s1lsa1j0k1aih9.png
