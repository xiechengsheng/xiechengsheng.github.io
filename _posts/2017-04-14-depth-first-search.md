---
layout:     post
title:      "深度优先搜索"
subtitle:   "dfs"
date:       "2017-04-14"
author:     "xiechengsheng"
header-img: "img/post-bg-22.jpg"
catalog: true
tags:
    - interview
---

# 典型题目
- T22. Generate Parentheses
生成括号序列（注意回溯的判断条件，在添加右边的括号的时候，条件是right<left）：
```cpp
    vector<string> generateParenthesis(int n) {
        vector<string> result;
        if(n<=0)
            return result;
        backtracking(result, "", 0, 0, n);
        return result;
    }

    void backtracking(vector<string> &result, string s, int left, int right, int n) {
        if(s.size()==2*n)
            result.push_back(s);
        else {
            if(left<n) {
                backtracking(result, s+"(", left+1, right, n);
            }
            if(right<left) {
                //当这个条件的if不成立的时候，向上回溯的层数不止一层，需要回溯到第一个if达到不满足的条件的之前的一个值
                //比如n=3，程序到达这里的时候left=right=3，向上发生回溯，是回溯到left=2，right=0的时候的情况，而不是到left=3，right=2，因为这只是回溯中的一种经历过程，先经过left=3，right=2，发现backtracking函数又结束了，再向上回溯，left=3，right=1，函数backtracking依旧结束，再回溯，left=3，right=0，函数依旧结束，回溯，left=2，right=0，发现第一个left<n的条件满足了，因此执行第一个条件
                backtracking(result, s+")", left, right+1, n);
            }
        }
    }
```

- T17. Letter Combinations of a Phone Number：按下手机的数字键，可以对应多少个字母的组合，按下“23”，组合是ad ae af bd be bf cd ce cf
```cpp
    vector<string> letterCombinations(string digits) {
        vector<string> result;
        int len=digits.size();
        if(len==0)
            return result;
        vector<string> characters{"","","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
        backtracking(result, "", 0, digits, characters);
        return result;
    }

    void backtracking(vector<string> &result, string s, int start, string digits, vector<string> characters) {
        if(s.size()==digits.size())
            result.push_back(s);
        else {
            //这里回溯的时候，判断的循环条件是每个按键对应的多个字母的长度
            string letters=characters[digits[start]-'0'];
            for(int i=0;i<letters.size();i++) {
                s.push_back(letters[i]);
                backtracking(result, s, start+1, digits, characters);
                s.pop_back();
            }
        }
    }
```

- T39. Combination Sum：将给定的一串不包含重复数字的数组，选其中任意多个数字加起来等于target，结果不能包括重复的集合
```cpp
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        sort(candidates.begin(),candidates.end());
        vector<vector<int>> results;
        //请注意vector<int> {}这种用法，直接当成参数使用
        backtraching(results, vector<int> {}, 0, candidates, target);
        return results;
    }

    void backtraching(vector<vector<int>> &results, vector<int> result, int start, vector<int> candidates, int remain) {
        //一般回溯函数里面只会存在两种判断条件，相等或者不等，现在是相等或者小于或者大于，因此是三种判断条件
        if(remain==0) {
            results.push_back(result);
        }
        else if(remain<0)
            return;
        else {
            for(int i=start;i<candidates.size();i++) {
                result.push_back(candidates[i]);
                //这里不能将i+1放到backtracking函数中，这样的话就不能包含可以重复选取一个数字了，只能选择一次这个数字
                backtraching(results, result, i, candidates, remain-candidates[i]);
                result.pop_back();
            }
        }
    }
```

- T131. Palindrome Partitioning：输入一个字符串，求出其所有的回文子字符串构成的多个序列
```cpp
    vector<vector<string>> partition(string s) {
        vector<vector<string>> results;
        backtracking(results, vector<string> {}, s, 0);
        return results;
    }

    //对于输入为"aab"的情况，首先考虑[0,0]"a"（剩下的[1,2]"ab"按照同样的方式进行处理，是一个递归的过程），其次是[0,1]"aa"，最后[0,2]"aab"
    void backtracking(vector<vector<string>> &results, vector<string> result, string s, int start) {
        if(start==s.size())
            results.push_back(result);
        else {
            for(int i=start;i<s.size();i++) {
                //只有满足回文条件的时候才会对整个字符串进行切割处理
                if(isPalindrome(s, start, i)) {
                    result.push_back(s.substr(start, i-start+1));
                    backtracking(results, result, s, i+1);
                    result.pop_back();
                }
            }
        }
    }

    bool isPalindrome(string s, int low, int high) {
        while(low<high) {
            if(s[low++]!=s[high--])
                return false;
        }
        return true;
    }
```

- T51. N-Queens：N皇后算法，暴力回溯计算出所有可以出现的解，格式背下来
```cpp
    vector<vector<string>> solveNQueens(int n) {
        vector<vector<string>> res;
        //初始化的时候没有放置皇后
        vector<string> nQueens(n, string(n,'.'));
        solveNQueens(res, nQueens, 0, n);
        return res;
    }
    
    //函数重载
    //暴力的方向：
    //...Q...，这样只需要考虑Q所在列之前有没有出现Q
    //这里的数组是可以优化成为一个一维数组的
    void solveNQueens(vector<vector<string>> &res, vector<string> &nQueens, int row, int n) {
        //已经进入到最后一行
        if(row==n) {
            res.push_back(nQueens);
            return;
        }
        for(int col=0;col<n;col++) {
            if(isValid(nQueens, row, col, n)) {
                nQueens[row][col]='Q';
                //继续深入向下一行进行判断
                solveNQueens(res, nQueens, row+1, n);
                nQueens[row][col]='.';
            }
        }
    }
    
    bool isValid(vector<string> &nQueens, int row, int col, int n) {
        //行本来只有一个皇后，不用检查
        //检查列
        for(int i=0;i<row;i++) {
            if(nQueens[i][col]=='Q')
                return false;
        }
        
        //检查45度方向（个人认为是135度）
        for(int i=row-1, j=col-1;i>=0&&j>=0;i--,j--) {
            if(nQueens[i][j]=='Q')
                return false;            
        }
        
        //检查135度方向（个人认为是225度）
        for(int i=row-1, j=col+1;i>=0&&j<n;i--,j++) {
            if(nQueens[i][j]=='Q')
                return false;            
        }        
        return true;
    }
```

- T52. N-Queens II：N皇后的所有摆法，不需要求解所有摆法
```cpp
    int totalNQueens(int n) {
        vector<bool> column(n, false);    //初始时候没有皇后占用位置，全为false
        vector<bool> dia45(2*n-1, false);
        vector<bool> dia135(2*n-1, false);
        int count = 0;
        totalNQueens(column, dia45, dia135, 0, n, count);
        return count;
    }
    
    //重载
    void totalNQueens(vector<bool> &column, vector<bool> &dia45, vector<bool> &dia135, int row, int n, int &count) {
        if(row==n) {
            count+=1;
            return;
        }
        for(int col=0;col<n;col++) {
            //可以放的条件
            if(!column[col]&&!dia45[col+row]&&!dia135[n-1-row+col]) {
                //假装放了皇后，直接把皇后占用的位置变成true，本来这些用于判断的数组应该是二维数组的，但是处于45度（个人认为是135度）和135度（个人认为是225度）的皇后是线性的，可以变成一维数组来解决！
                column[col]=dia45[col+row]=dia135[n-1-row+col]=true;
                totalNQueens(column, dia45, dia135, row+1, n, count);
                column[col]=dia45[col+row]=dia135[n-1-row+col]=false;
            }
        }
    }
```


# 总结
- 某些dp类型题目也可以使用dfs方法解决
- dfs的时间复杂度为指数级
- 有时候需要求解一些带有特定目标的解集，意思就是要将解集树中错误的路径剪枝
- 基本的代码实现框架：
    ```cpp
    void dfs(层数) {
        if(条件) {
            输出；
        }
        else {
            左子树的处理；
            dfs(层数+1)；
            右子树的处理；
            dfs(层数+1)；
        }
    }
    ```

# 参考
[彻头彻尾的理解回溯算法](http://blog.csdn.net/ffmpeg4976/article/details/45007439)    
[回溯问题java通解](https://discuss.leetcode.com/topic/46162/a-general-approach-to-backtracking-questions-in-java-subsets-permutations-combination-sum-palindrome-partioning)    
[C++的排列树解法](https://discuss.leetcode.com/topic/5881/my-elegant-recursive-c-solution-with-inline-explanation/32)    

