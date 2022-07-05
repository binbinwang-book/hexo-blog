---
title: LeetCode - 第一个错误的版本(Easy)
date: 2021-07-26 01:19
tags: [LeetCode,二分法]
categories: [算法]
---
# LeetCode - 第一个错误的版本(Easy)

难度：Easy

## 题目描述：

你是产品经理，目前正在带领一个团队开发新的产品。不幸的是，你的产品的最新版本没有通过质量检测。由于每个版本都是基于之前的版本开发的，所以错误的版本之后的所有版本都是错的。

假设你有 n 个版本 [1, 2, ..., n]，你想找出导致之后所有版本出错的第一个错误的版本。

你可以通过调用 bool isBadVersion(version) 接口来判断版本号 version 是否在单元测试中出错。实现一个函数来查找第一个错误的版本。你应该尽量减少对调用 API 的次数。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gstp4hce3aj308y08m74e.jpg)

## 我的题解

```
/**
 * Definition for isBadVersion()
 * 
 * @param {integer} version number
 * @return {boolean} whether the version is bad
 * isBadVersion = function(version) {
 *     ...
 * };
 */

/**
 * @param {function} isBadVersion()
 * @return {function}
 */
var solution = function(isBadVersion) {
    /**
     * @param {integer} n Total versions
     * @return {integer} The first bad version
     */
    return function(n) {
        var left = 1;
        var right = n;
        var middle = parseInt((left + right) / 2);

        if(left == right) {
            if(isBadVersion(left)){
                return left;
            }else{
                return 0;
            }
        }

        // 满足while的条件就一直执行，直到不满足条件
        while(left < right) {
            // 二分法通用代码
            if(isBadVersion(middle)) {
                right = middle - 1;
            }else{
                left = middle + 1;
            }
            middle = parseInt((left + right) / 2);

            if(left >= right) {
                // 业务逻辑
                if(isBadVersion(left)){
                    middle = left;
                }else{
                    middle = left + 1;
                }
                break;
            }
        }
        return middle;
    };
};
```

## 思考

这个题目我看了精彩题解，我拷贝如下：

```
class Solution {
public:
    int firstBadVersion(int n) {
        int left = 1, right = n;
        while (left < right) { // 循环直至区间左右端点相同
            int mid = left + (right - left) / 2; // 防止计算时溢出
            if (isBadVersion(mid)) {
                right = mid; // 答案在区间 [left, mid] 中
            } else {
                left = mid + 1; // 答案在区间 [mid+1, right] 中
            }
        }
        // 此时有 left == right，区间缩为一个点，即为答案
        return left;
    }
};

```

这个题解代码是比我写的清晰点，但实际上不够通用，如果换了一种二分查找的场景，这个代码就要比较大的改动。

二分查找的精髓其实在于下面：

```
        while(left < right) {
            // 二分法通用代码
            if(isBadVersion(middle)) {
                right = middle - 1;
            }else{
                left = middle + 1;
            }
            middle = parseInt((left + right) / 2);

            if(left >= right) {
                // 业务逻辑
            }
        }
```

查middle，middle没有命中，那么就根据 middle 移动 left 和 right，这属于通用的业务逻辑，以后凡是涉及到二分查找的，都可以用这样的流程代码。

最后说一句，这个题目的场景实际项目中也遇到过，当时也真的是用二分法来解决的，果然算法题都出自真实场景，只是大部分场景我还没遇到而已。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)