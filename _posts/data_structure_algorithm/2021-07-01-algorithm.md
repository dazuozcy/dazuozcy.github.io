---
layout: post
title: "算法"
author: dazuo
date: 2020-07-01 20:19:00 +0800
categories: [数据结构与算法]
tags: [Algorithm]
math: true
mermaid: true
---

https://programmercarl.com/0739.%E6%AF%8F%E6%97%A5%E6%B8%A9%E5%BA%A6.html#%E6%80%9D%E8%B7%AF



# 排序

## sort

```cpp
void sort (RandomAccessIterator first, RandomAccessIterator last, Compare comp);
```

```cpp
vector<int> v = {3,1,4,1,5,9,2,6};

sort(v.begin(),v.end()); // 升序
sort(v.begin(),v.end(), greater<>()); // 降序

// lamda表达式自定义
sort(v.begin(),v.end(), [](int a, int b){ return a > b;});

// 非内置类型
struct TestCase
{
    int first;
    int second;
};

int main() {
    vector<TestCase> v = {{4,2}, {1,3},{3,2},{3,1}};
    auto comp = [](TestCase &a, TestCase &b) {
        return a.first != b.first ? a.first < b.first : a.second < b.second;
    };
    sort(v.begin(), v.end(), comp);
    for(auto it = v.begin(); it != v.end(); ++it)
        cout<<(*it).first<<" " << (*it).second << endl;
    return 0;
}
/* 输出：
1 3
3 1
3 2
4 2
*/
```





[56. 合并区间](https://leetcode-cn.com/problems/merge-intervals/)

[539. 最小时间差](https://leetcode-cn.com/problems/minimum-time-difference/)

## 快速选择

[215. 数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

[347. 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

[462. 最少移动次数使数组元素相等 II](https://leetcode-cn.com/problems/minimum-moves-to-equal-array-elements-ii/)

## 堆排序

[215. 数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

[347. 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

## 拓扑排序

[207. 课程表](https://leetcode-cn.com/problems/course-schedule/)

## 桶排序

[451. 根据字符出现频率排序](https://leetcode-cn.com/problems/sort-characters-by-frequency/)



## 煎饼排序

[969. 煎饼排序](https://leetcode-cn.com/problems/pancake-sorting/)



## 计数排序

[1365. 有多少小于当前数字的数字](https://leetcode-cn.com/problems/how-many-numbers-are-smaller-than-the-current-number/)



## 自定义排序

[2191. 将杂乱无章的数字排序](https://leetcode-cn.com/problems/sort-the-jumbled-numbers/)



# 二分

[4. 寻找两个正序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)(hard)

[14. 最长公共前缀](https://leetcode-cn.com/problems/longest-common-prefix/)

[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

[35. 搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

[69. x 的平方根 ](https://leetcode-cn.com/problems/sqrtx/)

[74. 搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix/)

[153. 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

[154. 寻找旋转排序数组中的最小值 II](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/)

[278. 第一个错误的版本](https://leetcode-cn.com/problems/first-bad-version/)

[300. 最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

[354. 俄罗斯套娃信封问题](https://leetcode-cn.com/problems/russian-doll-envelopes/)

[374. 猜数字大小](https://leetcode-cn.com/problems/guess-number-higher-or-lower/)

[540. 有序数组中的单一元素](https://leetcode-cn.com/problems/single-element-in-a-sorted-array/)

[704. 二分查找](https://leetcode-cn.com/problems/binary-search/)

[1539. 第 k 个缺失的正整数](https://leetcode-cn.com/problems/kth-missing-positive-number/)

[2055. 蜡烛之间的盘子](https://leetcode-cn.com/problems/plates-between-candles/)

[面试题 08.03. 魔术索引](https://leetcode-cn.com/problems/magic-index-lcci/)



# 动态规划

https://leetcode-cn.com/problems/maximum-subarray/solution/dong-tai-gui-hua-fen-zhi-fa-python-dai-ma-java-dai/

[5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

[53. 最大子数组和](https://leetcode-cn.com/problems/maximum-subarray/)

[62. 不同路径](https://leetcode-cn.com/problems/unique-paths/)

[70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

[91. 解码方法](https://leetcode-cn.com/problems/decode-ways/)

[118. 杨辉三角](https://leetcode-cn.com/problems/pascals-triangle/)

[119. 杨辉三角 II](https://leetcode-cn.com/problems/pascals-triangle-ii/)

[120. 三角形最小路径和](https://leetcode-cn.com/problems/triangle/)

[121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

[122. 买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

[198. 打家劫舍](https://leetcode-cn.com/problems/house-robber/)

[213. 打家劫舍 II](https://leetcode-cn.com/problems/house-robber-ii/)

[338. 比特位计数](https://leetcode-cn.com/problems/counting-bits/)

[354. 俄罗斯套娃信封问题](https://leetcode-cn.com/problems/russian-doll-envelopes/)

[357. 计算各个位数不同的数字个数](https://leetcode-cn.com/problems/count-numbers-with-unique-digits/)

[416. 分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum/)

[509. 斐波那契数](https://leetcode-cn.com/problems/fibonacci-number/)

[583. 两个字符串的删除操作](https://leetcode-cn.com/problems/delete-operation-for-two-strings/)

[647. 回文子串](https://leetcode-cn.com/problems/palindromic-substrings/)

[698. 划分为k个相等的子集](https://leetcode-cn.com/problems/partition-to-k-equal-sum-subsets/)

[740. 删除并获得点数](https://leetcode-cn.com/problems/delete-and-earn/)

[746. 使用最小花费爬楼梯](https://leetcode-cn.com/problems/min-cost-climbing-stairs/)

[918. 环形子数组的最大和](https://leetcode-cn.com/problems/maximum-sum-circular-subarray/)

[978. 最长湍流子数组](https://leetcode-cn.com/problems/longest-turbulent-subarray/)

[1014. 最佳观光组合](https://leetcode-cn.com/problems/best-sightseeing-pair/)

[1137. 第 N 个泰波那契数](https://leetcode-cn.com/problems/n-th-tribonacci-number/)

[1525. 字符串的好分割数目](https://leetcode-cn.com/problems/number-of-good-ways-to-split-a-string/)

[2100. 适合打劫银行的日子](https://leetcode-cn.com/problems/find-good-days-to-rob-the-bank/)

# 贪心

[45. 跳跃游戏 II](https://leetcode-cn.com/problems/jump-game-ii/)

[55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

[134. 加油站](https://leetcode-cn.com/problems/gas-station/)

[135. 分发糖果](https://leetcode-cn.com/problems/candy/)

[300. 最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

[452. 用最少数量的箭引爆气球](https://leetcode-cn.com/problems/minimum-number-of-arrows-to-burst-balloons/)

[561. 数组拆分 I](https://leetcode-cn.com/problems/array-partition-i/)

[763. 划分字母区间](https://leetcode-cn.com/problems/partition-labels/)

[849. 到最近的人的最大距离](https://leetcode-cn.com/problems/maximize-distance-to-closest-person/)

[1007. 行相等的最少多米诺旋转](https://leetcode-cn.com/problems/minimum-domino-rotations-for-equal-row/)

[1328. 破坏回文串](https://leetcode-cn.com/problems/break-a-palindrome/)

[1338. 数组大小减半](https://leetcode-cn.com/problems/reduce-array-size-to-the-half/)

[1403. 非递增顺序的最小子序列](https://leetcode-cn.com/problems/minimum-subsequence-in-non-increasing-order/)

[1529. 最少的后缀翻转次数](https://leetcode-cn.com/problems/minimum-suffix-flips/)

[1551. 使数组中所有元素相等的最小操作数](https://leetcode-cn.com/problems/minimum-operations-to-make-array-equal/)

[1561. 你可以获得的最大硬币数目](https://leetcode-cn.com/problems/maximum-number-of-coins-you-can-get/)

[1665. 完成所有任务的最少初始能量](https://leetcode-cn.com/problems/minimum-initial-energy-to-finish-tasks/)

[2195. 向数组中追加 K 个整数](https://leetcode-cn.com/problems/append-k-integers-with-minimal-sum/)



# 回溯

[46. 全排列](https://leetcode-cn.com/problems/permutations/)

[78. 子集](https://leetcode-cn.com/problems/subsets/)

[698. 划分为k个相等的子集](https://leetcode-cn.com/problems/partition-to-k-equal-sum-subsets/)

[1239. 串联字符串的最大长度](https://leetcode-cn.com/problems/maximum-length-of-a-concatenated-string-with-unique-characters/)

[1253. 重构 2 行二进制矩阵](https://leetcode-cn.com/problems/reconstruct-a-2-row-binary-matrix/)

[2044. 统计按位或能得到最大值的子集数目](https://leetcode-cn.com/problems/count-number-of-maximum-bitwise-or-subsets/)



# DFS

[124. 二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

[130. 被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/)

[199. 二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/)

[200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

[207. 课程表](https://leetcode-cn.com/problems/course-schedule/)

[257. 二叉树的所有路径](https://leetcode-cn.com/problems/binary-tree-paths/)

[404. 左叶子之和](https://leetcode-cn.com/problems/sum-of-left-leaves/)

[617. 合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)

[2044. 统计按位或能得到最大值的子集数目](https://leetcode-cn.com/problems/count-number-of-maximum-bitwise-or-subsets/)

[2049. 统计最高分的节点数目](https://leetcode-cn.com/problems/count-nodes-with-the-highest-score/)

[剑指 Offer 17. 打印从1到最大的n位数](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

