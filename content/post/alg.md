---
title : "对算法笔试题的总结"
date: 2019-08-29T01:37:56+08:00
draft: false
tags: ["Java", "Algorithm"]
categories: ["Algorithm"]
author: "Yang Zhang"
---

算法笔试目前看来更多的是通过题目抽象出具体算法要求, 然后根据经验写出计算过程. 因此, 总结一些经典的方法和做过的题目还是很有必要的. 这篇博客用来拾遗用, 把做过的东西都捋一捋.
### 数组问题(Array Related)
#### 对撞指针

#### 滑动窗口
#### 寻找和为定值的多个数
### 字符串问题(String Related)
#### 子串问题
子串问题最常见的有求最长子串, 最短子串等问题, 与sequence不同, 子串的特点是连续性, 所以最优解一定是遍历一次字符串后就可以得到答案, 复杂度应为O(n). 例如76. Minimum Window Substring(string s 中包含string t的最小子串)


```Java
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

### 动态规划
动态规划属于算法题里较难的一部分,但是多加练习以后也能摸索出一些规律. 找出dp数组中的定义状态数组和状态转移方程是最关键的部分, 动态规划的几类标志性问题需要总结归纳一下.

- 0-1背包问题
- 最长递增子序列
- 最长公共子序列
- 最短编辑距离
- 格子取数
- 交替字符串

#### 0-1背包问题
题目为: 有一个容量为C的背包, 有0..n-1个物品, 每个物品重量为w(i), 价值为v(i), 求可容的最大价值的问题.

此类有容量限制的问题, 一般定义容量为状态, 具体的来说就是当容量为i时,它的最大价值应该为dp(i). 因此我们可以找到转化方程为(当前容量为i, 决定是否放入背包第j个物品)max(dp(i), dp(i- w(j))+ v(j))

值得注意的是, 这里状态的变化取决于两个变量. 因此dp的数组应该是二维的.
##### 相关的0-1背包问题
###### 当背包内的物品可以选择多次时
###### 
#### 最长上升子序列问题(LIS)
题目为: 给定一个数组, 找到最长的上升子序列.

首先, 序列与子串不同, 序列无需连续性, 因此无法通过遍历一次数组就找出问题的答案. 此时可以定义我们的状态为 dp(i)就是数组0..i区间里最长上升子序列的长度. 则转移方程就是把当前i位置的数值和每个之前的数值进行比较, 找出既上升又最大的那个. dp(i) = max(dp(i), dp(j)+1)

```Java
    public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        if (n < 2)
            return n;
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[i] > nums[j])
                    dp[i] = Math.max(dp[j]+1, dp[i]);
            }
        }
        return Arrays.stream(dp).max().getAsInt();
    }
```

拓展一下, 上升可以变化为任意比较性的条件, 所需要做的就是改变dp更新的condition.
#### 最长公共子序列(LCS)
题目为: 给两个字符串String a, String b, 找到最长公共子序列的长度.

分析此题的状态应该为a的前i个字符与b的前j个字符最长的公共子序列的长度, 因为最长的长度同时受这两个字符串的影响. 转移方程为 当a的第i个元素与b的第j个元素相同时, dp(i)(j) = max(dp(i-1)(j-1)+1, dp(i-1)(j), dp(i)(j-1)). 否则 dp(i)(j) = max(dp(i-1)(j), dp(i)(j-1))

```Java
    public int longestCommonSubsequence(String text1, String text2) {
        char[] arr1 = text1.toCharArray();
        char[] arr2 = text2.toCharArray();

        int[][] dp = new int[arr1.length+1][arr2.length+1];

        for (int i = 1; i <= arr1.length; i++) {
            for (int j = 1; j <= arr2.length; j++) {
                if(arr1[i-1] == arr2[j-1])
                    dp[i][j] = Math.max(dp[i-1][j], Math.max(dp[i][j-1], dp[i-1][j-1]+1));
                else
                    dp[i][j] = Math.max(dp[i-1][j],dp[i][j-1]);
            }
        }
        return dp[arr1.length][arr2.length];
    }
```

拓展一下, 相同的方式可能有所区别, 例如LeetCode 139号问题, 当单词匹配才算相同. 

#### 最短编辑距离
题目: 给定一个源串和目标串，能够对源串进行如下操作：1. 在给定位置上插入一个字符 2.替换任意字符 3.删除任意字符  写一个程序，返回最小操作数，使得对源串进行这些操作后等于目标串，源串和目标串的长度都小于2000。

首先定义状态, 假如令dp(i)(j) 表示源串S(0…i) 和目标串T(0…j)的最短编辑距离. 转移方程可以对S(i)和T(j)进行比较然后转移状态.
dp(i)(j) =min{
    dp(i-1)(j) + 1 , S(i)不在T(0…j)中
    dp(i-1)(j-1) + 1/0 , S(i)在T(j)
    dp(i)(j-1) + 1 , S(i)在T(0…j-1)中
}

#### 格子取数
题目: 在二维数组的左上方走到右下方, 只可以向下向右走,求最小的消耗.

首先也是定义状态, 令dp(i)(j)为从左上角走到二维数组中grid(i-1)(j-1)的最小消耗. 转移方程与最小编辑距离类似,上方的值和左方的值中小的那个加上当前值则为当前的最优解.

```Java
    public int minPathSum(int[][] grid) {
        int outSize = grid.length;
        int innerSize = grid[0].length;
        if (outSize == 1)
            return Arrays.stream(grid[0]).sum();

        int[][] dp = new int[outSize+1][innerSize+1];
        for (int i = 0; i <= innerSize; i++) {
            dp[0][i] = 0x7fff;
        }
        for (int i = 0; i <= outSize; i++) {
            dp[i][0] = 0x7fff;
        }
        dp[0][1] = 0;

        for (int i = 1; i <= outSize; i++) {
            for (int j = 1; j <= innerSize; j++) {
                dp[i][j] = Math.min(dp[i-1][j]+ grid[i-1][j-1], dp[i][j-1]+ grid[i-1][j-1]);
            }
        }
        return dp[outSize][innerSize];
    }
```

#### 交替字符串

题目: 输入三个字符串s1、s2和s3，判断第三个字符串s3是否由前两个字符串s1和s2交错而成，即不改变s1和s2中各个字符原有的相对顺序，例如当s1 = “aabcc”，s2 = “dbbca”，s3 = “aadbbcbcac”时，则输出true，但如果s3=“accabdbbca”，则输出false。

状态为dp(i)(j)代表s3(0...i+j-1)是否由s1(0...i-1)和s2(0...j-1)的字符组成. 

状态转移方程为

- 如果s1当前字符（即s1(i-1)）等于s3当前字符（即s3(i+j-1)），而且dp(i-1)(j)为真，那么可以取s1当前字符而忽略s2的情况，dp(i)(j)返回真；
- 如果s2当前字符等于s3当前字符，并且dp(i)(j-1)为真，那么可以取s2而忽略s1的情况，dp(i)(j)返回真，其它情况，dp(i)(j)返回假

```Java
    public boolean isInterleave(String s1, String s2, String s3){
        int n = s1.length(), m = s2.length(), s = s3.length();

        //如果长度不一致，则s3不可能由s1和s2交错组成
        if (n + m != s)
            return false;

        boolean[][]dp = new boolean[n + 1][m + 1];

        //在初始化边界时，我们认为空串可以由空串组成，因此dp[0][0]赋值为true。
        dp[0][0] = true;

        for (int i = 0; i < n + 1; i++){
            for (int j = 0; j < m + 1; j++){
                if ( dp[i][j] || (i - 1 >= 0 && dp[i - 1][j] == true &&
                    //取s1字符
                    s1.charAt(i - 1) == s3.charAt(i + j - 1)) ||

                    (j - 1 >= 0 && dp[i][j - 1] == true &&
                    //取s2字符
                    s2.charAt(j - 1) == s3.charAt(i + j - 1)) )

                    dp[i][j] = true;
                else
                    dp[i][j] = false;
            }
        }
        return dp[n][m];
    }
```

### 递归回溯法
其实基本所有的动态规划问题都可以用回溯法来解决, 只不过一个思路是从上到下, 一个思路是从下到上. 但是动态规划的局限之处是不能存储状态变化过程, 要存储的话需要得到最后结果以后回推就比较繁琐. 这时候用回溯法个人认为是一个更好的思路. 尤其解决排列和组合问题, 因为这两类问题都需要穷举(O(2^n)),因而递归的局限之处可以被遮盖. 话不多说开始总结.

#### 排列

题:(N0. 46) 给一个数组, 返回所有排列的可能

排列问题的关键是有序, 在代码中的表现是即使遍历到后面的index, 也需要考虑前面的元素. 这时需要纳入考虑范围的点是要排除那些已经考虑在内的点, 因此我们需要引入一个数组去记录当前的已经包括的数.

```java
import java.util.List;
import java.util.ArrayList;
import java.util.LinkedList;

public class Solution {

    private ArrayList<List<Integer>> res;
    private boolean[] used;

    public List<List<Integer>> permute(int[] nums) {

        res = new ArrayList<List<Integer>>();
        if(nums == null || nums.length == 0)
            return res;

        used = new boolean[nums.length];
        LinkedList<Integer> p = new LinkedList<Integer>();
        generatePermutation(nums, 0, p);

        return res;
    }

    // p中保存了一个有index-1个元素的排列。
    // 向这个排列的末尾添加第index个元素, 获得一个有index个元素的排列
    private void generatePermutation(int[] nums, int index, LinkedList<Integer> p){

        if(index == nums.length){
            res.add((LinkedList<Integer>)p.clone());
            return;
        }

        for(int i = 0 ; i < nums.length ; i ++)
            if(!used[i]){
                used[i] = true;
                p.addLast(nums[i]);
                generatePermutation(nums, index + 1, p );
                p.removeLast();
                used[i] = false;
            }

        return;
    }

    private static void printList(List<Integer> list){
        for(Integer e: list)
            System.out.print(e + " ");
        System.out.println();
    }

    public static void main(String[] args) {

        int[] nums = {1, 2, 3};
        List<List<Integer>> res = (new Solution()).permute(nums);
        for(List<Integer> list: res)
            printList(list);
    }
}
```

#### 排列问题拓展 -- 数组有重复数
当数组包含了重复的值时, 需要考虑更多的情况. 这时除了需要考虑当前的元素, 还需要考虑他前面的数值是否与当前元素相同. 因为如果相同的话, 就可以忽略当前节点, 因为前面一定已经分析了当前情况
```Java
public class Solution {
    public List<List<Integer>> permuteUnique(int[] nums) {
        List<List<Integer>> res = new ArrayList<List<Integer>>();
        if(nums==null || nums.length==0) return res;
        boolean[] used = new boolean[nums.length];
        List<Integer> list = new ArrayList<Integer>();
        Arrays.sort(nums);
        dfs(nums, used, list, res);
        return res;
    }

    public void dfs(int[] nums, boolean[] used, List<Integer> list, List<List<Integer>> res){
        if(list.size()==nums.length){
            res.add(new ArrayList<Integer>(list));
            return;
        }
        for(int i=0;i<nums.length;i++){
            if(used[i]) continue;
            if(i>0 &&nums[i-1]==nums[i] && !used[i-1]) continue;
            used[i]=true;
            list.add(nums[i]);
            dfs(nums,used,list,res);
            used[i]=false;
            list.remove(list.size()-1);
        }
    }
}
```

#### 组合
在组合问题中, 顺序是不重要的.

题:(No. 77) 给两个整数 n,k, 求1..n这n个数字中选出k个数字的所有组合.

在不断递归的时候, 我们当我们需要取第i个数的时候, 我们只需要考虑剩下i+1..n的组合, 前面的数无需考虑因为我们不前面的组合已经被考虑过了.

```Java
import java.util.List;
import java.util.ArrayList;
import java.util.LinkedList;

/// 77. Combinations
/// https://leetcode.com/problems/combinations/description/
/// 时间复杂度: O(n^k)
/// 空间复杂度: O(k)
public class Solution {

    private ArrayList<List<Integer>> res;

    public List<List<Integer>> combine(int n, int k) {

        res = new ArrayList<List<Integer>>();
        if(n <= 0 || k <= 0 || k > n)
            return res;

        LinkedList<Integer> c = new LinkedList<Integer>();
        generateCombinations(n, k, 1, c);

        return res;
    }

    // 求解C(n,k), 当前已经找到的组合存储在c中, 需要从start开始搜索新的元素
    private void generateCombinations(int n, int k, int start, LinkedList<Integer> c){

        if(c.size() == k){
            res.add((List<Integer>)c.clone());
            return;
        }

        for(int i = start ; i <= n ; i ++){
            c.addLast(i);
            generateCombinations(n, k, i + 1, c);
            c.removeLast();
        }

        return;
    }

    private static void printList(List<Integer> list){
        for(Integer e: list)
            System.out.print(e + " ");
        System.out.println();
    }

    public static void main(String[] args) {

        List<List<Integer>> res = (new Solution()).combine(4, 2);
        for(List<Integer> list: res)
            printList(list);
    }
}
```

练习题: No.39, No.40, No.216, No.78, No.90, No.401

#### 二维数组 - N皇后问题

N皇后问题其实有很多思路, 最简单的一个就是,首先保证每一行只有一个皇后, 然后判断接下来行里皇后的摆位. 其中判断的条件是皇后不可以与已经存在的皇后同行同列同对角线, 同行已经保证不可能, 因此我们需要三个数组去存储同列, 同对角线的皇后所在位置.

> 同对角线判断方法: 角度偏移45度的对角线上的数: 横纵坐标加起来是一个定值.  角度偏移-45度的对角线上的数: 横纵坐标减起来是一个定值. 对角线的个数有2*n-1个.

```Java
import java.util.Arrays;
import java.util.LinkedList;
import java.util.List;
import java.util.ArrayList;

/// 51. N-Queens
/// https://leetcode.com/problems/n-queens/description/
/// 时间复杂度: O(n^n)
/// 空间复杂度: O(n)
public class Solution {

    private boolean[] col;
    private boolean[] dia1;
    private boolean[] dia2;
    private ArrayList<List<String>> res;

    public List<List<String>> solveNQueens(int n) {

        res = new ArrayList<List<String>>();
        col = new boolean[n];
        dia1 = new boolean[2 * n - 1];
        dia2 = new boolean[2 * n - 1];

        LinkedList<Integer> row = new LinkedList<Integer>();
        putQueen(n, 0, row);

        return res;
    }

    // 尝试在一个n皇后问题中, 摆放第index行的皇后位置
    private void putQueen(int n, int index, LinkedList<Integer> row){

        if(index == n){
            res.add(generateBoard(n, row));
            return;
        }

        for(int i = 0 ; i < n ; i ++)
            // 尝试将第index行的皇后摆放在第i列
            if(!col[i] && !dia1[index + i] && !dia2[index - i + n - 1]){
                row.addLast(i);
                col[i] = true;
                dia1[index + i] = true;
                dia2[index - i + n - 1] = true;
                putQueen(n, index + 1, row);
                col[i] = false;
                dia1[index + i] = false;
                dia2[index - i + n - 1] = false;
                row.removeLast();
            }

        return;
    }

    private List<String> generateBoard(int n, LinkedList<Integer> row){

        assert row.size() == n;

        ArrayList<String> board = new ArrayList<String>();
        for(int i = 0 ; i < n ; i ++){
            char[] charArray = new char[n];
            Arrays.fill(charArray, '.');
            charArray[row.get(i)] = 'Q';
            board.add(new String(charArray));
        }
        return board;
    }

    private static void printBoard(List<String> board){
        for(String s: board)
            System.out.println(s);
        System.out.println();
    }

    public static void main(String[] args) {

        int n = 4;
        List<List<String>> res = (new Solution()).solveNQueens(n);
        for(List<String> board: res)
            printBoard(board);
    }
}
```