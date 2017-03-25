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

- lintcode：114. Unique Paths：机器人到达矩阵的右下角有多少条路径
```cpp
    int uniquePaths(int m, int n) {
        //自底向上的动归：dp[i][j]=dp[i][j-1]+dp[i-1][j]，表示当前坐标的路径数目
        if(!m||!n)
            return 0;
        vector<vector<int>> dp(m, vector<int>(n,0));
        for(int i=0;i<m;++i) {
            for(int j=0;j<n;++j) {
                if(i==0 || j==0) dp[i][j]=1;
                else dp[i][j]=dp[i][j-1]+dp[i-1][j];
            }
        }
        return dp[m-1][n-1];
    }
```

- lintcode：    115. Unique Paths II：机器人到达矩阵的右下角有多少条路径（带有障碍物）
```cpp
    int uniquePathsWithObstacles(vector<vector<int>> &obstacleGrid) {
        //自底向上的动归：dp[i][j]=dp[i][j-1]+dp[i-1][j]，表示当前坐标的路径数目
        //注意：初始状态的问题：如果i==0||j==0的情况下，有障碍物，那么后面的dp[i+1][0]||dp[0][j+1]直接是0，代表是不可达的
        
        if(obstacleGrid.empty()||obstacleGrid[0].empty())
            return 0;
        int m=obstacleGrid.size(), n=obstacleGrid[0].size();
        vector<vector<int>> dp(m, vector<int>(n,0));
        for(int j=0;j<n;++j) {
            if(obstacleGrid[0][j]) {
                dp[0][j] = 0;
                break;
            }
            else dp[0][j] = 1;
        }
        
        for(int i=0;i<m;++i) {
            if(obstacleGrid[i][0]) {
                dp[i][0] = 0;
                break;
            }
            else dp[i][0] = 1;
        }        
        
        for(int i=1;i<m;++i) {
            for(int j=1;j<n;++j) {
                if(obstacleGrid[i][j]) dp[i][j] = 0;
                else dp[i][j]=dp[i][j-1]+dp[i-1][j];
            }
        }
        return dp[m-1][n-1];
    }
```

- leetcode 139. Word Break：leetcode 能否由 ["leet", "code"]构成
```cpp
    bool wordBreak(string s, vector<string>& wordDict) {
        // 自底向上，单序列dp
        int max_len = 0;
        for(string str:wordDict) {
            max_len = max(max_len, (int)str.size());
        }
        
        int len=s.size();
        vector<bool> dp(len+1, false);    //dp[i]表示截止到第i个字符，能不能由wordDict构成
        dp[0]=true; //初始状态，s的第0个字符可以由wordDict构成

        for(int i=1;i<=len;++i) {
            for(int j=i-1;j>=0;--j) {
                if(i-j > max_len) break;    // 比最大的子字符串长度都要大，说明s[j:i]不可能由wordDict构成
                if(dp[j]&&find(wordDict.begin(), wordDict.end(), s.substr(j,i-j))!=wordDict.end()) { //s[j:i]可以由wordDict构成
                    dp[i]=true;
                    break;
                }
            }
        }
        return dp[len];
    }
```

- T300. Longest Increasing Subsequence：最长递增子序列
```cpp
    int lengthOfLIS(vector<int>& nums) {
        int len=nums.size();
        if(len<=1)
            return len;
        // 自底向上，单序列dp，dp[i]=max(dp[i], dp[j]+1)
        vector<int> dp(len,1);  // 截止到i时候的最长子序列
        dp[0]=1;        //初始状态=1
        
        for(int i=1;i<len;++i) {
            for(int j=0;j<i;++j) {
                if(nums[j]<nums[i])
                    dp[i]=max(dp[j]+1,dp[i]);   //更新最大值
            }
        }
        // return dp[len-1];
        
        int res=0;
        for(int i=0;i<len;++i)
            res=max(res, dp[i]);
        return res;
    }
```

- lintcode T77. Longest Common Subsequence：求两个字符串的最长公共子序列
```cpp
    int longestCommonSubsequence(string &A, string &B) {
        int m=A.size(), n=B.size();
        if(!m || !n)
            return 0;
        vector<vector<int>> dp(m, vector<int>(n,0));    //dp[i][j]表示A[i]与B[j]的LCS
        
        //状态初始化
        for(int i=0;i<m;++i) {
            if(A[i]==B[0])
                dp[i][0]=1;
        }
        for(int j=0;j<n;++j) {
            if(A[0]==B[j])
                dp[0][j]=1;
        }
        
        for(int i=1;i<m;++i) {
            for(int j=1;j<n;++j) {
                if(A[i]==B[j]) dp[i][j]=dp[i-1][j-1]+1;
                else dp[i][j]=max(dp[i-1][j],dp[i][j-1]);
            }
        }
        return dp[m-1][n-1];
    }
```

- leetcode T72 Edit Distance：求两个字符串的最小编辑距离
```cpp
    int minDistance(string word1, string word2) {
        int m=word1.size(), n=word2.size();
        //dp[i][j]表示word1的前i个字母和word2的前j个字母的minDistance
        vector<vector<int>> dp(m+1, vector<int>(n+1, 0));
        //初始状态
        for(int i=0;i<=m;++i) {
            dp[i][0] = i;
        }
        for(int j=0;j<=n;++j) {
            dp[0][j]=j;
        }
        
        //和LCS一样，分为word1[i]==word2[j]和word1[i]!=word2[j]两种情况进行状态转换
        for(int i=1;i<=m;++i) {
            for(int j=1;j<=n;++j) {
                if(word1[i-1]==word2[j-1]) {
                    dp[i][j]= dp[i-1][j-1];
                }
                else {
                    dp[i][j]=min(min(dp[i][j-1], dp[i-1][j]), dp[i-1][j-1])+1;
                }
            }
        }
        
        return dp[m][n];
    }
```


# 总结
- 动态规划总是需要首先找到递推关系，如果能够找到递推关系，那么直接使用暴力搜索也不失为一种好的解题思路；
- 多刷题，练出对动归方程的敏感度；



# 参考
[Dynamic Programming - 动态规划](https://algorithm.yuanbin.me/zh-hans/dynamic_programming/)    
[algorithm-exercise](https://github.com/billryan/algorithm-exercise)    
[LeetCode All in One 题目讲解汇总(持续更新中...)](http://www.cnblogs.com/grandyang/p/4606334.html)    
