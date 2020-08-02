---
title: LeetCode刷题记录-HOT 100
date: 2019-09-22 12:00:00
tags:
  - LeetCode
---

#### 1.两数之和

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer,Integer> map = new HashMap<>();
        int[] res = new int[2];
        for(int i = 0 ; i < nums.length ; ++ i){
            int temp = target - nums[i];
            
            if(map.containsKey(temp)){
                res[0] = map.get(temp);
                res[1] = i;
            }
            map.put(nums[i],i);
        }
        return res;
    }
}
```

题目可以通过暴力求解，时间复杂度为n方。这里用哈希表，用以空间换时间的方法，哈希表查找一趟的查找速度近乎为常数，但当查找冲突时时间会退到O(n)。

这里采用以一次遍历的哈希表，通过一次遍历将数组的元素放到表中，同时也以O(1)的速度查找target-nums[i]。

<!--more-->

#### 2.两数相加

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode l3 = new ListNode(0);
        ListNode p = l1,q = l2, cur = l3;
        int carry = 0;
        while(p != null || q != null){
            int x = (p == null)?0:p.val;
            int y = (q == null)?0:q.val;
            int sum = x + y + carry;
            carry = sum / 10;
            cur.next = new ListNode(sum % 10);
            cur = cur.next;
            if(p != null) p = p.next;
            if(q != null) q = q.next;
        }
        
        if(carry > 0){
            cur.next = new ListNode(carry);
        }
        return l3.next;
    }
}
```

#### 3.无重复字符的最长字串

给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。

输入: “abcabcbb”
输出: 3
解释: 因为无重复字符的最长子串是 “abc”，所以其长度为 3。

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        Map<Character,Integer> map = new HashMap<>();
        
        int max = 0 ,left = 0;
        for(int i = 0 ; i <s.length(); ++ i){
            if(map.containsKey(s.charAt(i))){
                left = Math.max(left,map.get(s.charAt(i))+1);
            }
            map.put(s.charAt(i),i);
            max = Math.max(max,i-left+1);
        }
        return max;
    }
}
```

维持一个滑动窗口，其实就是一个队列,比如例题中的 abcabcbb，进入这个队列（窗口）为 abc 满足题目要求，当再进入 a，队列变成了  abca，这时候不满足要求。所以，我们要移动这个队列。如何移动？我们只要把队列的左边的元素移出就行了，直到满足题目要求。一直维持这样的队列，找出队列出现最长的长度时候，求出解。

这里需要注意map.put如果放入重复的key元组会覆盖上一个key对应的value。

#### 5.最长回文子串

```java
class Solution {
    public String longestPalindrome(String s) {
       int start = 0,end = 0;
        for(int i = 0 ; i < s.length(); ++ i){
            int len1 = find(i,i,s);
            int len2 = find(i,i+1,s);
            
            int len = len1>len2?len1:len2;
            if(len > end - start){
                start = i - (len-1)/2;
                end = i + len/2 + 1;
            }
        }
        return s.substring(start,end);
    }
    public int find(int l,int r,String s){
        while(l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)){
            l--;
            r++;
        }
        return r-l-1;
    }
}
```

中心扩展算法，遍历每个元素，从中间开始扩展，注意有两种情况，一种是aba，一种是bbc

#### 15.三数之和

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        ArrayList list = new ArrayList<List<Integer>>();
        Arrays.sort(nums);
        
        for(int i = 0 ; i < nums.length-2; ++ i){
            int j = i + 1, k = nums.length - 1;
            
            if(nums[i] > 0)break;
            
            if(i > 0 && nums[i] == nums[i-1]) continue;
            
            while(j < k){
                if(nums[i] + nums[j] + nums[k] < 0){
                    while(j < k && nums[j] == nums[++j]);
                }else if(nums[i] + nums[j] + nums[k] > 0){
                    while(j < k && nums[k] == nums[--k]);
                }else{
                    ArrayList tmp = new ArrayList<Integer>();
                    tmp.add(nums[i]);tmp.add(nums[j]);tmp.add(nums[k]);
                    list.add(tmp);
                    while(j < k && nums[k] == nums[--k]);
                    while(j < k && nums[j] == nums[++j]);
                }
            }
        
        }
        
        return list;
    }
}
```

#### 17.电话号码的组合

```java
class Solution {
    public List<String> letterCombinations(String digits) {
        if(digits == null ||digits.length() == 0)
            return new ArrayList<String>();
        Map map = new HashMap<Character,String>();
        map.put('1',"");map.put('2',"abc");map.put('3',"def");map.put('4',"ghi");map.put('5',"jkl");
        map.put('6',"mno");map.put('7',"pqrs");map.put('8',"tuv");map.put('9',"wxyz");
        List<String> result = new LinkedList<>();
        
        combination(digits,"",result,0,map);
        
        return result;
    }
    
    public void combination(String digits,String subStr,List<String> result,int index,Map<Character,String>map){
        if(index == digits.length()){
            result.add(subStr);
            return ;
        }
        
        String letters = map.get(digits.charAt(index));
        for(int i = 0 ; i < letters.length(); ++ i){
            combination(digits,subStr+letters.charAt(i),result,index+1,map);
        }
        
    }
}
```

#### 22.括号生成

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String>list = new ArrayList<>();
        
        backtrack(list,"",0,0,n);
        
        return list;
    }
    
    public void backtrack(List<String>list,String subString,int open,int close,int n){
        if(open+close == 2 * n){
            list.add(subString);
            return;
        }
        //如果我们还剩一个位置，我们可以开始放一个左括号。 如果它不超过左括号的数量，我们可以放一个右括号
        if(open < n)
            backtrack(list,subString+"(",open+1,close,n);
        if(open > close)
            backtrack(list,subString+")",open,close+1,n);
    }
}
```

#### 31.下一个排列

```java
class Solution {
    public void nextPermutation(int[] nums) {
        if (nums == null || nums.length == 0) return;
        int firstIndex = -1;
        for (int i = nums.length - 2; i >= 0; i--) {
            if (nums[i] < nums[i + 1]) {
                firstIndex = i;
                break;
            }
        }
        if (firstIndex == -1) {
            reverse(nums, 0, nums.length - 1);
            return;
        }
        int secondIndex = -1;
        for (int i = nums.length - 1; i >= 0; i--) {
            if (nums[i] > nums[firstIndex]) {
                secondIndex = i;
                break;
            }
        }
        swap(nums, firstIndex, secondIndex);
        reverse(nums, firstIndex + 1, nums.length - 1);
        return;

    }

    private void reverse(int[] nums, int i, int j) {
        while (i < j) {
            swap(nums, i++, j--);
        }
    }

    private void swap(int[] nums, int i, int i1) {
        int tmp = nums[i];
        nums[i] = nums[i1];
        nums[i1] = tmp;
    }
}
```

![1556953035922.png](https://pic.leetcode-cn.com/4169e8e0c8b4d71d4d32b4f50b09a57c0ea951cb4bdbd16a785d5847959e261f-1556953035922.png)

这道题是根据 维基百科 翻译过来：

```java
先找出最大的索引 k 满足 nums[k] < nums[k+1]，如果不存在，就翻转整个数组；
再找出另一个最大索引 l 满足 nums[l] > nums[k]；
交换 nums[l] 和 nums[k]；
最后翻转 nums[k+1:]。
```

举个例子：

比如 nums = [1,2,7,4,3,1]，下一个排列是什么？

我们找到第一个最大索引是 nums[1] = 2

再找到第二个最大索引是 nums[5] = 3

交换，nums = [1,3,7,4,2,1];

翻转，nums = [1,3,1,2,4,7]

完毕!

#### 33.搜索旋转排序数组

```java
class Solution {
    public int search(int[] nums, int target) {
        int left = 0,right = nums.length-1;
        
        while(left < right){
            int mid = (left + right)/2;
            if((nums[0] <= target) ^ (target <= nums[mid]) ^ (nums[mid] < nums[0]))
                left = mid + 1;
            else
                right = mid;
        }
        
        return left == right && nums[left] == target? left:-1;
    }
}
```

注意到原数组为有限制的有序数组（除了在某个点会突然下降外均为升序数组）

if nums[0] <= nums[I] 那么 nums[0] 到 nums[i] 为有序数组,那么当 nums[0] <= target <= nums[i] 时我们应该在 0−i0-i0−i 范围内查找；

if nums[i] < nums[0] 那么在 0−i0-i0−i 区间的某个点处发生了下降（旋转），那么 I+1I+1I+1  到最后一个数字的区间为有序数组，并且所有的数字都是小于 nums[0] 且大于 nums[i]，当target不属于 nums[0] 到  nums[i] 时（target <= nums[i] < nums[0] or nums[i] < nums[0]  <= target），我们应该在 0−i0-i0−i 区间内查找。
上述三种情况可以总结如下：

nums[0] <= target <= nums[i]
           target <= nums[i] < nums[0]
                     nums[i] < nums[0] <= target

所以我们进行三项判断：

(nums[0] <= target)， (target <= nums[i]) ，(nums[i] < nums[0])，现在我们想知道这三项中有哪两项为真（明显这三项不可能均为真或均为假（因为这三项可能已经包含了所有情况））

所以我们现在只需要区别出这三项中有两项为真还是只有一项为真。

使用 “异或” 操作可以轻松的得到上述结果（两项为真时异或结果为假，一项为真时异或结果为真，可以画真值表进行验证）

之后我们通过二分查找不断做小 target 可能位于的区间直到 low==high，此时如果 nums[low]==target 则找到了，如果不等则说明该数组里没有此项。

#### 39.组合总和

```java
class Solution {
    
    public void backtrack(int[] candidates,int dist,int index,List<List<Integer>> res,Stack <Integer> s){
        if(dist < 0){
            return ;
        }
        if(dist == 0){
            res.add(new ArrayList<>(s));
            return ;
        }
        for(int i = index ; i < candidates.length; ++ i){//注意这里的index，防止重复
            if(dist >= candidates[i]){
                s.push(candidates[i]);
                backtrack(candidates,dist-candidates[i],i,res,s);
                s.pop();//不合适的加上需要剔除
            }
                
        }
    }
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> res = new ArrayList<>();
        Stack <Integer>s = new Stack<>();
        
        backtrack(candidates,target,0,res,s);
        
        return res;
    }
}
```

#### 46.全排列

```java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        Stack<Integer> s = new Stack<>();
        List<List<Integer>> res = new ArrayList<>();
        int []visited = new int[nums.length];
        backtrack(nums,visited,s,res);
        return res;
        
    }
    public void backtrack(int[] nums,int[]visited,Stack<Integer>s,List<List<Integer>>res){
        if(s.size() == nums.length){
            res.add(new ArrayList<Integer>(s));
            return;
        }
        
        for(int i = 0; i < nums.length ; ++ i){
            
            if(visited[i] == 1)continue;
            visited[i] = 1;
            s.push(nums[i]);
            backtrack(nums,visited,s,res);
            s.pop();
            visited[i] = 0;
        }
    }
}
```

#### 48.旋转图像

```java
class Solution {
    public void swap(int[][]matrix,int i,int j){
        int tmp = matrix[i][j];
        matrix[i][j] = matrix[j][i];
        matrix[j][i] = tmp;
        
    }
    public void rotate(int[][] matrix) {
        for(int i = 0 ; i < matrix.length; ++ i){
            for(int j = 0 ; j < i; ++ j){
                int tmp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = tmp;
            }
        }
        
        for(int i = 0 ; i < matrix.length; ++ i){
            for(int j = 0 ; j < matrix.length / 2; ++ j){
                int tmp = matrix[i][j];
                matrix[i][j] = matrix[i][matrix[0].length-j-1];
                matrix[i][matrix[0].length-j-1] = tmp;
            }
        }
    }
}
```

先转置再镜像

#### 49.字母异位词分组

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String,List> map = new HashMap<>();
        
        for(String s : strs){
            char[] c = s.toCharArray();//字符串转字符数组
            Arrays.sort(c);
            String str = String.valueOf(c);
            if(!map.containsKey(str))
                map.put(str,new ArrayList<String>());
            map.get(str).add(s);
        }
        
        return new ArrayList(map.values());//用Collection构造ArrayList
    }
}
```

#### 53.最大子序和

```java
class Solution {
 public int maxSubArray(int[] nums) {
        int sum = 0,res = nums[0];
        for(int num : nums){
            if(sum <= 0 )
                sum = num;
            else
                sum += num;
            
            res = Math.max(res,sum);
        }
        return res;
    }
}
```

sum为**当前**最大连续子序和，res为最终结果。

刚开始的时候sum为0，res为第一个数，遍历一次数组，只有在当前子序和大于0的时候才有可能成为最终的最大子序和，而当前子序和小于0的时候从头开始计子序和，即令当前子序和等于元素。每次读一个元素后判断当前子序和是否大于最终结果。

#### 55.跳跃游戏

```java
class Solution {
    public boolean canJump(int[] nums) {
        int k = 0 ;
        for(int i = 0; i < nums.length; ++ i){
            if(i > k)return false;
            k = Math.max(k,i + nums[i]);
        }
            return true;
    }
}
```

如果某一个作为起跳点的格子可以跳跃的距离是3，那么表示后面3个格子都可以作为起跳点。
可以对每一个能作为起跳点的格子都尝试跳一次，把能跳到最远的距离不断更新。
如果可以一直跳到最后，就成功了。

#### 56.合并区间

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        if(intervals == null || intervals.length == 0)
            return new int[0][];
        Arrays.sort(intervals,(a,b)->a[0]-b[0]);
        List<int[]>res = new ArrayList<>();
        int i = 0;
        while(i < intervals.length){
            int left = intervals[i][0];
            int right = intervals[i][1];
            
            while(i < intervals.length && right >= intervals[i][0]){
                right = Math.max(intervals[i][1],right);
                i++;
            }
            
            res.add(new int[]{left,right});
        }
        
        return res.toArray(new int[0][]);
    }
}
```

#### 62.不同路径

```java
class Solution {
    public int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];
        
        for(int i = 0 ; i < m ; ++ i)
            dp[i][0] = 1;
        for(int i = 0 ; i < n ; ++ i)
            dp[0][i] = 1;
        for(int i = 1 ; i < m; ++ i){
            for(int j = 1;j  < n; ++ j){
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
        
        return dp[m-1][n-1];
    }
}
```

动态规划：我们令 dp[i][j] 是到达 i, j 最多路径

动态方程：dp[i][j] = dp[i-1][j] + dp[i][j-1]

注意，对于第一行 dp[0][j]，或者第一列 dp[i][0]，由于都是在边界，所以只能为 1

#### 64.最小路径和

```java
class Solution {
    public int minPathSum(int[][] grid) {
        int[][] dp = new int[grid.length][grid[0].length];
        
        for(int i = 0 ; i < grid.length; ++ i){
            for(int j = 0 ; j < grid[0].length; ++ j){
                if(i == 0 && j == 0)
                    dp[i][j] = grid[i][j];
                else if(i != 0 && j == 0)
                    dp[i][j] = grid[i][j] + dp[i-1][j];
                else if(i == 0 && j != 0)
                    dp[i][j] = grid[i][j] + dp[i][j-1];
                else
                    dp[i][j] = grid[i][j] + Math.min(dp[i-1][j],dp[i][j-1]);
            }
        }
        
        return dp[grid.length-1][grid[0].length-1];
    }
}
```

动态规划：我们新建一个额外的 dpdpdp 数组，与原矩阵大小相同。在这个矩阵中，dp(i,j)dp(i, j)dp(i,j) 表示从坐标 (i,j)(i, j)(i,j) 到右下角的最小路径权值。我们初始化右下角的 dpdpdp  值为对应的原矩阵值，然后去填整个矩阵，对于每个元素考虑移动到右边或者下面，因此获得最小路径和我们有如下递推公式：

dp(i,j)=grid(i,j)+min⁡(dp(i+1,j),dp(i,j+1))

#### 70.爬楼梯

```java
class Solution {
    public int climbStairs(int n) {
        int[] dp = new int[n];
        if(n == 1)return 1;
        if(n == 2)return 2;
        dp[0] = 1;dp[1] = 2;
        for(int i = 2 ; i < n ; i++){
            dp[i] = dp[i-1] + dp[i-2];
        }
        
        return dp[n-1];
    }
}
```

不难发现，这个问题可以被分解为一些包含最优子结构的子问题，即它的最优解可以从其子问题的最优解来有效地构建，我们可以使用动态规划来解决这一问题。

第 i 阶可以由以下两种方法得到：

在第 (i−1)阶后向上爬一阶。

在第 (i−2) 阶后向上爬 2阶。

所以到达第 i阶的方法总数就是到第 (i−1)阶和第 (i−2)阶的方法数之和。

令 dp[i]表示能到达第 i阶的方法总数：

dp[i]=dp[i−1]+dp[i−2]

**这里注意到达n-2层的时候是走一次俩步直接到n，而不是走两个一步，因为一步的情况已经包含在了n-1层里**

#### 78.子集

```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        List<Integer> tmp = new ArrayList<>();
        
        backtrack(res,tmp,nums,0);
        
        return res;
    }
    public void backtrack(List<List<Integer>> res,List<Integer>tmp,int[] nums,int index){
        res.add(new ArrayList<>(tmp));
        for(int i = index ; i < nums.length; ++ i){
            tmp.add(nums[i]);
            backtrack(res,tmp,nums,i+1);
            tmp.remove(tmp.size()-1);
        }
    }
}
```

#### 79.单词搜索

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        
        int row = board.length,col = board[0].length;
        for(int i = 0 ; i < row; ++ i){
            for(int j = 0 ; j < col; ++ j){
                if(board[i][j] == word.charAt(0)){
                    
                    boolean[][] visited = new boolean[row][col];
                    if(backtrack(board,word,0,i,j,visited) == true)
                        return true;
                }
            }
        }
        
        return false;
    }
    public boolean backtrack(char[][] board,String word,int index,int i,int j,boolean[][] visited){
        if(index == word.length()){
            return true;
        }
        if(i < 0 || i >= board.length || j < 0 || j >= board[0].length || board[i][j] != word.charAt(index) || visited[i][j])
            return false;
        
        visited[i][j] = true;
        
        if(backtrack(board,word,index+1,i+1,j,visited) || backtrack(board,word,index+1,i,j+1,visited)
          || backtrack(board,word,index+1,i-1,j,visited) || backtrack(board,word,index+1,i,j-1,visited))
            return true;
        
        visited[i][j] = false;
        
        return false;
    }
}
```

#### 94.二叉树的中序遍历

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    class ColorNode {
        TreeNode node;
        String color;
        
        public ColorNode(TreeNode node,String color){
            this.node = node;
            this.color = color;
        }
    }
    public List<Integer> inorderTraversal(TreeNode root) {
        if(root == null) return new ArrayList<Integer>();
            
        List<Integer> res = new ArrayList<>();
        Stack<ColorNode> stack = new Stack<>();
        stack.push(new ColorNode(root,"white"));
        
        while(!stack.empty()){
            ColorNode cn = stack.pop();
            
            if(cn.color.equals("white")){
                if(cn.node.right != null) stack.push(new ColorNode(cn.node.right,"white"));
                stack.push(new ColorNode(cn.node,"gray"));
                if(cn.node.left != null)stack.push(new ColorNode(cn.node.left,"white"));
            }else{
                res.add(cn.node.val);
            }
        }
        
        return res;
    }
}
```

使用了一种颜色标记法

使用颜色标记节点的状态，新节点为白色，已访问的节点为灰色。
如果遇到的节点为白色，则将其标记为灰色，然后将其右子节点、自身、左子节点依次入栈。
如果遇到的节点为灰色，则将节点的值输出。

这个方法很好，递归与非递归的记忆方法相同，只是顺序相反。如中序遍历：递归是左根右，非递归是右根左；先序遍历：递归左右根，非递归根右左。后序类似。

#### 96.不同的二叉搜索树

```java
class Solution {
    public int numTrees(int n) {
        int dp[] = new int[n+1];
        dp[0] = 1;dp[1] = 1;
        for(int i = 2 ; i <= n; ++ i){
            for(int j = 0; j < i; ++ j){
                dp[i] += dp[j] * dp[i-j-1];
            }
        }
        return dp[n];
    }
}
```

动态规划，假设n个节点存在二叉排序树的个数是G(n)，令f(i)为以i为根的二叉搜索树的个数，则

G(n)=f(1)+f(2)+f(3)+f(4)+…+f(n)

当i为根节点时，其左子树节点个数为i-1个，右子树节点为n-i，则

f(i)=G(i−1)∗G(n−i)

综合两个公式可以得到 卡特兰数 公式

G(n)=G(0)∗G(n−1)+G(1)∗(n−2)+…+G(n−1)∗G(0)

#### 98.验证二叉搜索树

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    
    public boolean isValidBST(TreeNode root) {
        if(root == null)return true;
        if(root != null && root.left == null && root.right == null) return true;
        List<Integer> list = new ArrayList<>();
        backtrack(root,list);
        
        for(int i = 1 ; i < list.size(); ++ i){
            if(list.get(i-1) >= list.get(i))
                return false;
        }
        
        return true;
    }
    public void backtrack(TreeNode root,List<Integer> list){
        
        if(root == null)
            return;
        backtrack(root.left,list);
        list.add(root.val);
        backtrack(root.right,list);
    }
}
```

因为二叉搜索树中序遍历是**绝对递增的,不存在相等的情况。**所以我们可以中序遍历判断前一数是否小于后一个数.

#### 101.对称二叉树

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isMirror(TreeNode lnode,TreeNode rnode){
        if(lnode == null && rnode == null)return true;
        if(lnode == null || rnode == null)return false;
        
        if(lnode.val == rnode.val && isMirror(lnode.left,rnode.right) && isMirror(lnode.right,rnode.left))
            return true;
        
        return false;
    }
    public boolean isSymmetric(TreeNode root) {
        if(root == null)return true;
        return isMirror(root.left,root.right);
    }
}
```

#### 102.二叉树层次遍历

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if(root == null)return new ArrayList<>();
        List<List<Integer>> res = new ArrayList<>();
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
        while(!q.isEmpty()){
            
            int size = q.size();
            List<Integer> list = new ArrayList<>();
            for(int i = 0; i < size; i ++){
                TreeNode node = q.remove();
                
                list.add(node.val);
                if(node.left != null) q.offer(node.left);
                if(node.right != null) q.offer(node.right);
            }
            
            res.add(list);
        }
        
        return res;
    }
}
```

#### 105.从前序与中序遍历序列构造二叉树

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    
    public TreeNode helper(int[]preorder,int p_start,int p_end,int[] inorder,int i_start,int i_end){
        if(p_start == p_end){
            return null;
        }
        
        int root_val = preorder[p_start];
        TreeNode root = new TreeNode(root_val);
        int i_root_index = 0;
        for(int i = i_start;i < i_end; ++ i){
            if(inorder[i] == root_val){
                i_root_index = i;
                break;
            }
        }
        
        int left_num = i_root_index - i_start;
        root.left = helper(preorder,p_start+1,p_start+1+left_num,inorder,i_start,i_root_index);
        root.right = helper(preorder,p_start+1+left_num,p_end,inorder,i_root_index+1,i_end); 
        
        return root;
    }
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        
        return helper(preorder,0,preorder.length,inorder,0,inorder.length);
    }
}
```

#### 114.二叉树展开为链表

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public void flatten(TreeNode root) {
        
        while(root != null){
            if(root.left == null){
                root = root.right;
            }else{
                TreeNode pre = root.left;
                while(pre.right != null){
                    pre = pre.right;
                }
                
                pre.right = root.right;
                root.right = root.left;
                root.left = null;
                root = root.right;
            }
        }
    }
}
```

#### 136.只出现一次的数字

```java
class Solution {
    public int singleNumber(int[] nums) {
        int ans = nums[0];
        
        for(int i = 1; i < nums.length; ++ i){
            ans = ans ^ nums[i];
        }
            
        return ans;
    }
}
```

异或可用于交换，可以用于一堆数字异或后找到只出现一次的数字，相同数字异或为0，0和任何数异或均为原数字。一堆数字异或不区分运算顺序。

#### 139.单词拆分

```java
class Solution {
    public boolean backtrack(String s,HashSet<String>wordDict,int start,Boolean[] flag){
        if(start == s.length()){
            return true;
        }
        if(flag[start] != null){
            return flag[start];
        }
        for(int end = start+1 ; end <= s.length(); ++ end){
            if(wordDict.contains(s.substring(start,end)) && backtrack(s,wordDict,end,flag))
                return flag[start] = true;
        }
        return flag[start] = false;
    }
    public boolean wordBreak(String s, List<String> wordDict) {
        HashSet<String> set = new HashSet<>(wordDict);
        
        return backtrack(s,set,0,new Boolean[s.length()]);
    }
}
```

运用了剪枝

#### 146.LRU缓存机制

```java
class Node{
        protected int key;
        protected int val;
        protected Node pre;
        protected Node next;
        
        public Node(int key,int val){
            this.key = key;
            this.val = val;
        }
    }
    class NodeList{
        private Node head;
        private Node tail;
        private int size;
        
        public NodeList(){
            head = new Node(0,0);
            tail = new Node(0,0);
            head.next = tail;
            tail.pre = head;
            size = 0;
        }
        public void addToHead(Node node){
            node.next = head.next;
            node.pre = head;
            head.next.pre = node;
            head.next = node;
            size ++;
        }
        
        public Node removeTail(){
            if(size == 0) return null;
            Node node = tail.pre;
            node.pre.next = tail;
            tail.pre = node.pre;
            size --;
            
            return node;
            
        }
        public void remove(Node node){
            node.pre.next = node.next;
            node.next.pre = node.pre;
            size --;
        }
        public int getSize(){
            return size;
        }
    }
class LRUCache {
    private NodeList list;
    private Map<Integer,Node> map;
    private int capacity;
    public LRUCache(int capacity) {
        list = new NodeList();
        map = new HashMap<>();
        this.capacity = capacity;
    }
    
    public int get(int key) {
        
        if(!map.containsKey(key))
            return -1;
        
        Node node = map.get(key);
        list.remove(node);
        list.addToHead(node);
        //put(key,node.val);
        return node.val;
    }
    
    public void put(int key, int value) {
    
        Node node = new Node(key,value);
        if(map.containsKey(key)){
            list.remove(map.get(key));
            list.addToHead(node);
            map.put(key,node);
        }else{
            if(list.getSize() >= capacity) {
                node = list.removeTail();
                map.remove(node.key);
            }
            node = new Node(key,value);
            list.addToHead(node);
            map.put(key,node);
        }
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```

#### 148.排序链表

```java
class Solution {
    public ListNode sortList(ListNode head){
        if(head == null)
            return null;
        return mergeSort(head);
    }
    public ListNode mergeSort(ListNode head){
        ListNode dummyHead = new ListNode(0);
        dummyHead.next = head;
        int length = 0;
        ListNode cur = head;
        while(cur != null){
            length ++;
            cur = cur.next;
        }
        
        for(int size = 1; size < length; size <<= 1){
            cur = dummyHead.next;
            ListNode tail = dummyHead;
            while(cur != null){
                 
                ListNode left = cur;
                ListNode right = cut(cur,size);
                cur = cut(right,size);
                
                tail.next = Merge(left,right);
                while(tail.next != null){
                    tail = tail.next;
                }
            }
        }   
        return dummyHead.next;
    }
    
    public ListNode cut(ListNode head,int size){
        ListNode cur = head;
        while(--size > 0 && cur != null){
            cur = cur.next;
        }
        if(cur == null) return null;
        ListNode next = cur.next;
        cur.next = null;
        return next;
    }
    
    public ListNode Merge(ListNode left,ListNode right){
        ListNode dummyHead = new ListNode(0);
        ListNode cur = dummyHead;
        while(left != null && right != null){
            if(left.val < right.val){
                cur.next = left;
                left = left.next;
                cur = cur.next;
            }else{
                cur.next = right;
                right = right.next;
                cur = cur.next;
            }
        }
        if(left == null){
            cur.next = right;
        }
        if(right == null){
            cur.next = left;
        }
        
        return dummyHead.next;
    }
    
}
```

#### 152.乘积最大子序列

```java
class Solution {
    public int maxProduct(int[] nums) {
        int max = Integer.MIN_VALUE, imax = 1, imin = 1;
        for(int i=0; i<nums.length; i++){
            if(nums[i] < 0){ 
              int tmp = imax;
              imax = imin;
              imin = tmp;
            }
            imax = Math.max(imax*nums[i], nums[i]);
            imin = Math.min(imin*nums[i], nums[i]);
            
            max = Math.max(max, imax);
        }
        return max;
    }
}
```

#### 155.最小栈

```java
class MinStack {

    Stack<Integer> stack;
    Stack<Integer> helper;
    /** initialize your data structure here. */
    public MinStack() {
        stack = new Stack<>();
        helper = new Stack<>();
    }
    
    public void push(int x) {
        stack.add(x);
        
        if(helper.isEmpty() || helper.peek() >= x) {
            helper.add(x);
        }else {
            helper.add(helper.peek());
        }
    }
    
    public void pop() {
        
        stack.pop();
        helper.pop();
    }
    
    public int top() {
        
        return stack.peek();
    }
    
    public int getMin() {
        
        return helper.peek();
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(x);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.getMin();
 */
```

保持辅助栈的栈顶元素始终是当前栈的最小元素

#### 160.相交链表

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    if (headA == null || headB == null) return null;
    ListNode pA = headA, pB = headB;
    while (pA != pB) {
        pA = pA == null ? headB : pA.next;
        pB = pB == null ? headA : pB.next;
    }
    return pA;
}
```

#### 198.打家劫舍

```java
class Solution {
    public int rob(int[] nums) {
        if(nums == null || nums.length == 0)return 0;
        if(nums.length == 1)return nums[0];
        if(nums.length == 2)return Math.max(nums[0],nums[1]);
        
        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        dp[1] = Math.max(nums[0],nums[1]);
        for(int i = 2; i < nums.length; i++) {
            dp[i] = Math.max(dp[i-1],dp[i-2]+nums[i]);
        }
        
        return dp[nums.length-1];
    }
}
```

注意这里的dp[1]不是nums[1]，当有只有两个房子可以偷的时候，可以偷的最大值Math.max(nums[0],nums[1])

#### 200.岛屿数量

```java
class Solution {
    public int numIslands(char[][] grid) {
        if(grid == null || grid.length == 0)
            return 0;
        int sum = 0;
        for(int i = 0 ; i < grid.length; i++){
            for(int j = 0 ; j < grid[0].length; j++){
                if(grid[i][j] == '1'){
                    bfs(grid,i,j);
                    sum ++;
                }
            }
        }
        
        return sum;
    }
    public void bfs(char[][]grid,int i,int j){
        
        if(i < 0 || j >= grid[0].length || j < 0 || i >= grid.length || grid[i][j] == '0'){
            return;
        }
        
        grid[i][j] = '0';
        bfs(grid,i,j+1);
        bfs(grid,i+1,j);
        bfs(grid,i,j-1);
        bfs(grid,i-1,j);
    }
}
```

#### 207.课程表

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        //拓扑排序
        if(numCourses <= 0)return false;
        if(prerequisites.length == 0)return true;
        int[] inDegree = new int[numCourses]; 
        for(int[]p : prerequisites){
            inDegree[p[0]]++;
        }
        
        Queue<Integer> queue = new LinkedList<>();
        for(int i = 0 ; i < numCourses; i++){
            if(inDegree[i] == 0)
                queue.add(i);
        }
        List<Integer>res = new LinkedList<>();
        while(!queue.isEmpty()){
            int num = queue.poll();
            res.add(num);
            for(int[] p : prerequisites){
                if(p[1] == num){
                    inDegree[p[0]]--;
                    if(inDegree[p[0]] == 0)
                        queue.add(p[0]);
                }
            }
        }
        return res.size() == numCourses;
    }
}
```

#### 215.数组中的第K个最大元素

```java
class Solution {
        public int findKthLargest(int[] nums, int k) {
        int max = Integer.MIN_VALUE,min = Integer.MAX_VALUE;
            
        for(int num:nums){
            if(max < num)max = num;
            if(min > num)min = num;
        }
        int dis = max - min;
            
        int[] bucket = new int[dis+1];
        for (int i = 0 ; i < nums.length; i++){
            bucket[nums[i]-min] ++;
        }
        for(int i = dis; i >= 0; i--){
            if(k > 0)
                k -= bucket[i];
            if(k <= 0)
                return min+i;
        }
            
        return 0;
    }
}
```

#### 221.最大正方形

```java
class Solution {
    public int maximalSquare(char[][] matrix) {
        //动态规划
        int row = matrix.length;
        if(row == 0)return 0;
        int col = matrix[0].length;
        int[][] dp = new int[row+1][col+1];
        
        int maxLength = Integer.MIN_VALUE;
        for (int i = 1; i <= row; i++){
            for (int j = 1; j <= col; j++){
                if(matrix[i-1][j-1] == '1'){
                    dp[i][j] = Math.min(Math.min(dp[i-1][j],dp[i][j-1]),dp[i-1][j-1])+1;
                    if(maxLength < dp[i][j])
                        maxLength = dp[i][j];
                }
            }
        }
        return maxLength * maxLength;
    }
}
```

#### 234.回文链表

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isPalindrome(ListNode head) {
        if(head == null)return true;
        if(head.next == null)return true;
        ListNode slow = head,fast = head;
        int step = 0;
        while(fast != null && fast.next != null){
            slow = slow.next;
            fast = fast.next.next;
            step++;
        }
        
        ListNode second = slow.next,three;
        while(second != null){
            three = second.next;
            second.next = slow;
            slow = second;
            second = three;
        }
        
        while(step > 0){
            if(slow.val != head.val)
                return false;
            slow = slow.next;
            head = head.next;
            step --;
        }
        return true;
    }
}
```

#### 236.二叉树的最近祖先

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode result = null;
    public int backtrack(TreeNode root,TreeNode p,TreeNode q){
        if(root == null)
            return 0;
        
        int left = backtrack(root.left,p,q);
        int right = backtrack(root.right,p,q);
        int mid = (root == p || root == q)? 1:0;
        
        if(left+right+mid >= 2)
            this.result = root; 
        
        return mid+left+right>=1? 1:0;
    }
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        
        backtrack(root,p,q);
        return this.result;
    }
}
```

#### 238.除自身以外数组的乘积

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        //乘积 = 左边的乘积 * 右边的乘积
        int tmp = 1;
        int[] mul = new int[nums.length];
        for(int i = 0; i < nums.length; i++){
            mul[i] = tmp;
            tmp *= nums[i];
        }
        tmp = 1;
        for(int i = nums.length-1; i >= 0; i--){
            mul[i] *= tmp;
            tmp *= nums[i];
        }
        return mul;
    }
}
```

#### 240.搜索二维矩阵II

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        if(matrix == null || matrix.length == 0)return false;
        int i = 0,j = matrix[0].length-1;
        while(i < matrix.length && j >= 0){
            if(target < matrix[i][j])
                j--;
            else if(target > matrix[i][j])
                i++;
            else
                return true;
        }
        return false;
    }
}
```

#### 276.完全平方数

```java
class Solution {
    /*动态规划。
    定义一个函数f(n)表示我们要求的解。f(n)的求解过程为：
    f(n) = 1 + min{f(n-1^2), f(n-2^2), f(n-3^2), 
    f(n-4^2), ... , f(n-k^2) //(k为满足k^2<=n的最大的k)}*/
    public int numSquares(int n) {
        int[] dp = new int[n+1];
        for (int i = 1; i <= n; i++){
            dp[i] = i;//最坏情况下全是1
            for (int j = 1; i - j*j >= 0; j++){
                dp[i] = Math.min(dp[i],dp[i-j*j]+1);
            }
        }
        return dp[n];
    }
}
```

#### 287.寻找重复数

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int left = 1,right = nums.length-1;
        
        while(left < right){
            int count = 0;
            int mid = (left+right)/2;
            for(int num:nums){
                if(num <= mid)
                    count ++;
            }
            if(count > mid)
                right = mid;
            else
                left = mid + 1;   
        }
        return left;
    }
}
```

#### 300.最长上升子序列

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        //动态规划
        int[]dp = new int[nums.length];
        
        Arrays.fill(dp,1);
        for (int i = 0; i < nums.length; i++) {
            
            for (int j = 0; j < i; j++) {
                if(nums[i] > nums[j]){
                    dp[i] = Math.max(dp[i],dp[j]+1);
                }
            }
        }
        int res = 0;
        for(int i = 0; i < nums.length; i++){
            res = Math.max(dp[i],res);
        }
        
        return res;
    }
}
```

#### 309.最佳买卖股票时机含冷冻期

```java
class Solution {
    public int maxProfit(int[] prices) {
        int dp_i_0 = 0,dp_i_1 = Integer.MIN_VALUE;
        int pre_i_0 = 0;
        for(int i = 0; i < prices.length; i++){
            int tmp = dp_i_0;
            dp_i_0 = Math.max(dp_i_0, dp_i_1 + prices[i]);
            dp_i_1 = Math.max(dp_i_1, pre_i_0 - prices[i]);
            pre_i_0 = tmp;
        }
        
        return dp_i_0;
    }
}
```

#### 322.零钱兑换

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        
        int[] dp = new int[amount+1];
        Arrays.fill(dp,amount+1);
        dp[0] = 0;
        
        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if(coin <= i){
                    dp[i] = Math.min(dp[i],dp[i-coin]+1);
                }
            }
        }
        return dp[amount] == amount+1? -1:dp[amount];
    }
}
```

#### 337.打家劫舍III

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] backtrack(TreeNode root){
        int[] res = new int[2];
        if(root == null){
            return res;
        }
        
        int[] left = backtrack(root.left);
        int[] right = backtrack(root.right);
        res[0] = Math.max(left[0],left[1]) + Math.max(right[0],right[1]);
        res[1] = left[0] + right[0] + root.val;
        
        return res;
        
    }
    public int rob(TreeNode root) {
        int[] res = backtrack(root);
        return Math.max(res[0],res[1]);
    }
}
```

#### 338.比特位计数

```java
class Solution {
    public int[] countBits(int num) {
        int[] dp = new int[num+1];
        for(int i = 0; i <= num; i ++){
            if(i % 2 == 0)
                dp[i] = dp[i/2];
            else
                dp[i] = dp[i-1] + 1;
        }
        
        return dp;
    }
}
```

#### 406.根据身高重建队列

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        //先排序后插入
        //高个子先站好，矮个子插到K位置上，前面正好K个高个子
        //后面的矮个子会打乱前面矮个子的位置，但不会改变它的K值因为K只和大于等于自己的人相关
        Arrays.sort(people,(p1,p2)->p1[0]==p2[0]?p1[1]-p2[1]:p2[0]-p1[0]);
        List<int[]> res = new LinkedList<>();
        for(int[] p : people){
            res.add(p[1],p);
        }
        
        return res.toArray(new int[res.size()][2]);
    }
}
```

toArray里面有想转换的类型和大小，如果大小比res的大小要小，转换出来就是res的原大小，如果传的大小比

#### 494.目标和

```java
class Solution {
    public int findTargetSumWays(int[] nums, int S) {
        int sum = 0;
        
        for(int num : nums)
            sum += num;
        if((sum + S) % 2 == 1 || sum < S)return 0;
        int target = (sum + S)/2;
        int[] dp = new int[target+1];
        dp[0] = 1;
        for(int num : nums){
            for(int i = target; i >= num; i--){
                dp[i] += dp[i-num];//这些解法里面走一步即可到达dp[i]
            }
        }
        return dp[target];
    }
}
```

#### 538.把二叉搜索树转换为累加树

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    int sum;
    public TreeNode backtrack(TreeNode root){
        if(root == null)
            return root;
        
        backtrack(root.right);
        int tmp = root.val;
        root.val += sum;
        sum += tmp;
        backtrack(root.left);
        
        return root;
    }
    public TreeNode convertBST(TreeNode root) {
        return backtrack(root);
        
    }
}
```

#### 560.和为K的子数组

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        if(nums == null)return 0;
        
        Map<Integer,Integer> map = new HashMap<>();
        int sum = 0,res = 0;
        map.put(0,1);//2-0,3-1
        for (int i = 0;i < nums.length; i++) {
            sum += nums[i];
            
            if(map.containsKey(sum - k))
                res += map.get(sum-k);
            map.put(sum,map.getOrDefault(sum,0)+1);
        }
        
        return res;
    }
}
```

#### 621.任务调度器

```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        
        int[] counts = new int[26];
        
        for(int i = 0; i < tasks.length; i++)
            counts[tasks[i] - 'A']++;
        Arrays.sort(counts);
        int maxCount = 0;
        for(int i = 25; i>= 0; i--)
            if(counts[i] == counts[25])
                maxCount++;
            else
                break;
        
        return Math.max((counts[25] - 1)*(n+1) + maxCount,tasks.length);
    }
}
```

#### 647.回文子串

```java
class Solution {
    public int countSubstrings(String s) {
        int res = 0;
        
        for(int i = 0; i < s.length(); i++){
            res += countSubString(s,i,i);
            res += countSubString(s,i,i+1);
        }
        
        return res;
    }
    
    public int countSubString(String s,int start,int end){
        int count = 0;
        while(start >= 0 && end < s.length() && s.charAt(start--) == s.charAt(end++))
            count++;
        
        return count;
    }
}
```

#### 739.每日温度

```java
class Solution {
   public int[] dailyTemperatures(int[] T) {
    int length = T.length;
    int[] result = new int[length];

    //从右向左遍历
    for (int i = length - 2; i >= 0; i--) {
        // j+= result[j]是利用已经有的结果进行跳跃
        for (int j = i + 1; j < length; j+= result[j]) {
            if (T[j] > T[i]) {
                result[i] = j - i;
                break;
            } else if (result[j] == 0) { //遇到0表示后面不会有更大的值，那当然当前值就应该也为0
                result[i] = 0;
                break;
            }
        }
    }

    return result;
	}
}
```