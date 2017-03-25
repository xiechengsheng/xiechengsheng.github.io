---
layout:     post
title:      "理解动态规划"
subtitle:   "dp"
date:       "2017-03-25"
author:     "xiechengsheng"
header-img: "img/post-bg-21.jpg"
catalog: true
tags:
    - interview
---

# 动态规划分类
- 动态规划可以分为：单序列和多序列
1. 单序列：状态通常定义为：数组前 i 个位置, 数字, 字母 或者 以第i个为... 返回结果通常为数组的最后一个元素；
    - 按照动态规划的四要素，此类题目可从以下四个角度分析。
        1. State: f[i] 前i个位置/数字/字母...
        2. Function: f[i] = f[i-1]... 找递推关系
        3. Initialization: 根据题意进行必要的初始化
        4. Answer: f[n-1]
2. 双序列(DP_Two_Sequence)：一般有两个数组或者两个字符串，计算其匹配关系。双序列中常用二维数组表示状态转移关系；

# 典型DP题
- T120. Triangle
```cpp
    int minimumTotal(vector<vector<int>>& triangle) {
        //双序列题目
        // //1.自底向上
        // //方程：f(x,y)=min(f(x+1,y),f(x+1,y+1))+trangle[x][y]
        // vector<vector<int>> dp(triangle);   //拷贝一份数组
        // if(triangle.size()==0)
        //     return 0;
        // if(triangle.size()==1)
        //     return triangle[0][0];
        // int len=triangle.size();
        // for(int i=len-2;i>=0;--i){
        //     for(int j=0;j<triangle[i].size();++j)
        //         dp[i][j]=min(dp[i+1][j],dp[i+1][j+1])+triangle[i][j];
        // }
        // return dp[0][0];
        //2.自顶向下
        //方程：f(x,y)=min(f(x-1,y),f(x-1,y-1))+trangle[x][y]
        vector<vector<int>> dp(triangle);   //拷贝一份数组
        if(triangle.size()==0)
            return 0;
        if(triangle.size()==1)
            return triangle[0][0];
        int len=triangle.size();
        for(int i=1;i<len;++i) {
            for(int j=0;j<triangle[i].size();++j) {
                if(j==0) dp[i][j]=dp[i-1][j]+triangle[i][j];
                else if(j==i) dp[i][j]=dp[i-1][j-1]+triangle[i][j];
                else dp[i][j]=min(dp[i-1][j],dp[i-1][j-1])+triangle[i][j];
            }
        }
        //从最后一行挑选最小值
        int dp_size=dp.size();
        int result=INT_MAX;
        for(int i=0;i<dp_size;++i) {
            if(result>dp[dp_size-1][i]) result=dp[dp_size-1][i];
        }
        return result;
    }
```

- lintcode：110. Minimum Path Sum经过二维矩阵grid的最短路径：
```cpp
    int minPathSum(vector<vector<int>> &grid) {
        // 状态转移方程：dp[i][j]=min(dp[i-1][j]+dp[i][j-1])+grid[i][j]      //自底向上，dp[i][j]表示到达该位置的最短的和

        if(grid.empty()||grid[0].empty())
            return 0;
        int m=grid.size(), n=grid[0].size();
        vector<vector<int>> dp(m, vector<int>(n,0));   //构造dp数组
        
        for(int i=0;i<m;++i) {
            for(int j=0;j<n;++j) {
                if(i==0 && j==0) dp[i][j]=grid[i][j];
                else if(i==0 && j>=1) dp[i][j]=dp[i][j-1]+grid[i][j];
                else if(i>=1 && j==0) dp[i][j]=dp[i-1][j]+grid[i][j];
                else {
                    int temp = dp[i-1][j]>dp[i][j-1]?dp[i][j-1]:dp[i-1][j];
                    dp[i][j]= temp+grid[i][j];
                }
            }
        }
        return dp[m-1][n-1];
    }
```



# 总结
- 动态规划总是需要首先找到递推关系，如果能够找到递推关系，那么直接使用暴力搜索也不失为一种好的解题思路；



# 参考
[Dynamic Programming - 动态规划](https://algorithm.yuanbin.me/zh-hans/dynamic_programming/)    
[algorithm-exercise](https://github.com/billryan/algorithm-exercise)    
[LeetCode All in One 题目讲解汇总(持续更新中...)](http://www.cnblogs.com/grandyang/p/4606334.html)    
