---
title: X数之和图文详解
top: false
cover: false
toc: true
mathjax: true
date: 2021-07-12 15:56:02
password:
summary:
categories: 数据结构与算法
tags: 
	- 数组
	- 极客时间
---

## 1.两数之和

### 1.1 题目描述

> 给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 的那 两个 整数，并返回它们的数组下标。
>
> 你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。
>
> 你可以按任意顺序返回答案。

### 1.2 思路及复杂度分析

首先大家直观的想法，会考虑使用两层循环分别遍历数组，并将其相加的和与目标值相比较。这样的解法其平均时间复杂度为O(N2)，着实不怎么优雅。

那优化的空间在哪呢？既然通过第一层已经确定了一个值，那满足要求的值就呼之欲出了，也就是说问题转化为**在一个数组中快速地找目标值**。首先给出答案，通过**基于散列表的哈希查找的平均时间复杂度为O(1)**，我们简要的聊下哈希查找；

> **散列表**（**Hash table**，也叫**哈希表**），是根据[键](https://zh.wikipedia.org/wiki/鍵)（Key）而直接访问内存储存位置的[数据结构](https://zh.wikipedia.org/wiki/数据结构)。也就是说，它通过计算一个关于键值的函数，将所需查询的数据[映射](https://zh.wikipedia.org/wiki/映射)到表中一个位置来访问记录，这加快了查找速度。这个映射函数称做[散列函数](https://zh.wikipedia.org/wiki/散列函数)，存放记录的数组称做**散列表**。

也就是说在哈希表中，键值和元素存在唯一的映射关系，那这个键值如何获取呢？需要通过**哈希函数**，这里只简要讨论**除留余数法**——

若已知整个哈希表的最大长度m，可以取一个不大于m的数p，然后对该关键字key做取余运算。如： key = key % p。

很显然，通过哈希函数得到的值是有可能冲突的，当发生**哈希冲突**时，就需要为引起冲突的元素重新确定哈希值，这里我们介绍解决哈希冲突的方法之一——**线性探测法**

在线性探测法中，当遇到冲突时，从发生冲突位置起，每次 +1，向右探测，直到不发生哈希冲突为止。

### 1.3 图解演示

图片来自于https://leetcode-cn.com/problems/two-sum/solution/yi-miao-jiu-neng-gao-dong-de-dong-tu-jie-cp6x/

![两数之和](https://pic.leetcode-cn.com/1610336380-bKlBYP-file_1610336383206)

### 1.4 代码演示

```java
class Solution {
  public int[] twoSum(int[] nums, int target) {
    //创建一个hashmap
    Map<Integer, Integer> map = new HashMap<>();
    for(int i = 0; i< nums.length; i++) {
      //如果哈希表中存在目标元素，就将其封装返回
      if(map.containsKey(target - nums[i])) {
        return new int[] {map.get(target-nums[i]),i};
	  }
      //不存在的话，就将其加入哈希表，这里好理解的写法是在哈希表初始化以后就将数组元素全部添加进去，但放在这个位置，动态添加，时间复杂度更低
      map.put(nums[i], i);
    }
    throw new IllegalArgumentException("No two sum solution");
  }
}
```

> 日拱一卒，功不唐捐。

## 2.三数之和

### 2.1 题目描述

> 给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有和为 0 且不重复的三元组。
>
> 注意：答案中**不可以包含重复的三元组**。

### 2.2 思路及复杂度分析

三数之和无疑是两数之和的进阶了，迷惑的是，居然还有四数之和，其实呢，掌握了三数之和，四数之和是很easy的，接下来我们言归正传——

直接想到**排序+双指针**的朋友还是有的，但我个人并不推荐直接上手就使用优化的思路。大多数朋友的思维是没有那么跳跃的，比如我。我们不妨**从最直观的方法写起**，一步步优化，方可有的放矢。

最直观的无非是使用**三重循环**，首先排序，每层循环的值不重复，再对三个数的加和与目标值做判断，就OK了，但讲真确实不优雅。

三层循环的时间复杂度O(N3)太感人了，能不能降低循环的层数，也就是说**在数组中找第三个值的时间复杂度要为O(1),**在两数之和中使用的**哈希表是一个可取的思路**，但为了拓宽视野，我们考虑别的方式，很显然，确定了第一个值，那么剩下的**两个数就满足加和等于target-nums[i]**,所以一层循环解决两个数就变的可行了，**第三个数从数组末尾开始倒序访问**即可。

到这里，核心的思路已经解决了。但是仍旧需要额外做一些修修补补，也就是**特判**。首先如果数组的长度小于3或者第一个元素的值就大于target,那就没有继续的必要了。其次，同一层循环如果碰到相同的值，应该跳过。最后，为了避免第二个数和第三个数重复，可以设置左指针的位置小于右指针的位置。

### 2.3 动图解析

大家可以移步leetcode题解区(大佬牛逼)：https://leetcode-cn.com/problems/3sum/solution/yi-miao-jiu-neng-kan-dong-de-dong-tu-jie-unfp/

### 2.4 代码演示

```java
class Solution {
  public List<List<Integer>> threeSum(int[] nums) {
    //创建二维数组
    List<List<Integer>> lists = new ArrayList<>();
    //将数据先排序
    Arrays.sort(nums);
    //一层循环，确保当前循环的数和上一次循环的数不同
    if(nums.length<=2 || nums[0]>0)
      return lists;
    int i = 0;
    for(;i<nums.length-2;i++){
      if(nums[i]>0)
        break;
      if(i > 0 && nums[i] == nums[i-1])
        continue;
      int target = -nums[i];
      //声明双指针的位置
      int left = i+1;
      int right = nums.length-1;
      while(left < right){
        if(nums[left] + nums[right] == target) {
          lists.add(new ArrayList<>(Arrays.asList(nums[i], nums[left], nums[right])));
          //这里是许多朋友容易困惑的地方，合理的顺序是下面这样的，可以理解为下面这部分都是为下一次的操作数做准备。
          left++;
          right--;
          while (left<right&&nums[left] == nums[left - 1])
            left++;
          while (left<right&&nums[right] == nums[right + 1])
            right--;
        }else if(nums[left] + nums[right] > target){
          right--;
        }else{
          left++;
        }
      }
    }
    return lists;
  }
}
```

## 3.四数之和

可能很多人看到这里比较火大，这还有完没完了~ 要不你给我整个N数之和。。。

其实呢，我也觉得到四数之和这里已经没有新的解法，技巧融合进去了，就姑且当做是三数之和的练习题咯，水个题数(嘿嘿)。好了，进入正题——

### 3.1 题目描述

> 给定一个包含 n 个整数的数组 nums 和一个目标值 target，判断 nums 中是否存在四个元素 a，b，c 和 d ，使得 a + b + c + d 的值与 target 相等？找出所有满足条件且不重复的四元组。
>
> 注意：
>
> 答案中不可以包含重复的四元组。

### 3.2 算法思路和复杂度分析

好吧，从思路上来说的确毫无新意，但解法中需要注意的细节还是很多的，这里我们只描述容易出错的细节——

1.  当用双指针实现时，所用时间会下降很多，究其根本，是因为双指针类似于二分查找，时间复杂度为O(logN),而哈希表的查找时的时间复杂度是O(1).
2.  预估的最小值大于目标值，则break,而最大值小于目标值，需要使用continue.这个坑在初次写的时候义无反顾的跳了。。
3.  在比较重复元素时，判断条件之一是当前层循环变量的值大于上一层循环变量的值加一。通俗的说，这个值需要取它有可能取的值，这么说是有点绕了，但自己打一遍代码就觉得非常有必要了。

### 3.3 动图演示

请移步leetcode大佬做的动图：https://leetcode-cn.com/problems/4sum/solution/yi-miao-jiu-neng-gao-dong-de-duo-ge-qiu-ya50w/

### 3.4 代码展示

```java
class Solution {
  public List<List<Integer>> fourSum(int[] nums, int target) {
    List<List<Integer>> lists = new ArrayList<>();
    //排好序
    Arrays.sort(nums);
    //特判，数组长度小于4返回空
    if(nums.length<4)
      return lists;
    //将数组元素添加到哈希表中
    HashMap<Integer, Integer> map = new HashMap<>();
    for(int i=0;i<nums.length;i++){
      map.put(nums[i],i);
    }
    //遍历数组中的所有元素
    int i = 0;
    int n = nums.length;
    for(;i<nums.length-3;i++){
      //当出现重复元素，则跳过开始下一次循环
      if(i>0 && nums[i]==nums[i-1])
        continue;
      //特判，即该层循环中最大的和小于target或者最小的和大于target
      if(nums[i]+nums[i+1]+nums[i+2]+nums[i+3]>target)
       	  break;
      if(nums[i]+nums[n-1]+nums[n-2]+nums[n-3]<target)
          continue;
      int j = i+1;
      //从第二个元素开始遍历数组
      for(;j<nums.length-2;j++){
          //当出现重复元素，则开始下一次循环
        if(j>i+1 && nums[j]==nums[j-1])
          continue;
        //特判，即当前循环中最大的和小于target或者最小的和大于target
        if(nums[i]+nums[j]+nums[j+1]+nums[j+2]>target)
          break;
        if(nums[i]+nums[j]+nums[n-1]+nums[n-2]<target)
          continue;
        int k = j+1;
        for(;k<nums.length-1;k++){
          //当出现重复元素，则开始下一次循环
          if(k>j+1 && nums[k]==nums[k-1])
            continue;
          int num = target-nums[i]-nums[j]-nums[k];
          if(map.containsKey(num) && map.get(num)>k){
            lists.add(new ArrayList<>(Arrays.asList(nums[i],nums[j],nums[k],target-nums[i]-nums[j]-nums[k])));
         }
        }
      }
    }
    return lists;
  }
}

```

------

> 山高路远，静水深流