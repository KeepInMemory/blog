---
title: LeetCode刷题记录-精选TOP面试题
date: 2019-10-23 12:00:00
tags:
  - LeetCode
---

#### 34.在排序数组中查找元素的第一个和最后一个位置

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int[] res = new int[2];
        Arrays.fill(res,-1);
        if(nums.length == 0)return res;
        int left = 0,right = nums.length-1;
        
        while(left < right){//寻找左侧边界
            int mid = left + (right-left)/2;
            if(nums[mid] == target)
                right = mid;
            else if(nums[mid] > target)
                right = mid;
            else
                left = mid + 1;
        }
        if(left == nums.length)res[0] = -1;
        else res[0] = (nums[left] == target? left:-1);
        
        left = 0;right = nums.length;
        while(left < right){//寻找右侧边界
            int mid = left + (right-left)/2;
            if(nums[mid] == target)
                left = mid + 1;
            else if(nums[mid] > target)
                right = mid;
            else 
                left = mid + 1;
        }
        if(left == 0)res[0] = -1;
        else res[1] = (nums[left-1] == target? left-1:-1);
        
        
        return res;
    }
}
```

#### 50.Pow(x,n)

```java
class Solution {
    public double fastPow(double x,int n){
        if(n == 0)return 1;
        
        double half = fastPow(x,n/2);
        if(n % 2 == 0)
            return half * half;
        else 
            return half * half * x;
    }
    public double myPow(double x, int n) {
        if(n < 0){
            return fastPow(1/x,-1*n);
        }else{
            return fastPow(x,n);
        }
    }
}
```

#### 73.矩阵置零

```java
class Solution {
    
    public void setZeroes(int[][] matrix) {
        if(matrix.length == 0)return;
        boolean col_zero = false,row_zero = false;
        //第一列是否有零
        for(int i = 0; i < matrix.length; i++)
            if(matrix[i][0] == 0){
                col_zero = true;
                break;
            }
        //第一行是否有零
        for(int j = 0;j < matrix[0].length; j++)
            if(matrix[0][j] == 0){
                row_zero = true;
                break;
            }
        
        //把第一行第一列作为标志位
        for(int i = 1;i < matrix.length; i++){
            for(int j = 1; j < matrix[0].length; j++){
                if(matrix[i][j] == 0){
                    matrix[0][j] = 0;
                    matrix[i][0] = 0;
                }
            }
        }
        
        //从第二行第二列开始置零
        for(int i = 1 ; i < matrix.length; i++){
            if(matrix[i][0] == 0){
                for(int j = 1; j < matrix[0].length; j++)
                    matrix[i][j] = 0;
            }
        }
        
        for(int j = 1; j < matrix[0].length; j++){
            if(matrix[0][j] == 0){
                for(int i = 1; i < matrix.length; i++)
                    matrix[i][j] = 0;
            }
        }
        
        //将第一列置零
        if(col_zero){
            for(int i = 0; i < matrix.length; i++){
                matrix[i][0] = 0;
            }
        }
        //将第一行置零
        if(row_zero){
            for(int j = 0; j < matrix[0].length; j++){
                matrix[0][j] = 0;
            }
        }
    }
}
```

#### 103.二叉树的锯齿形层次遍历

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
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        if(root == null)return new LinkedList<List<Integer>>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        int num = 1;
        
        List res = new LinkedList<List<Integer>>();
        List<Integer>list;
        while(!queue.isEmpty()){
            int size = queue.size();
            list = new LinkedList<>();
            while(size-- > 0){
                TreeNode node = queue.poll();
                if(num % 2 == 0)
                    list.add(0,node.val);
                else
                    list.add(node.val);
                if(node.left != null)queue.add(node.left);
                if(node.right != null)queue.add(node.right);
            }
            res.add(list);
            num ++;
        }
        
        return res;
    }
}
```

#### 108.将有序数组转化为二叉搜索树

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
    public TreeNode sortedArrayToBST(int[] nums) {
        return backtrack(nums,0,nums.length-1);//左闭右闭
    }
    public TreeNode backtrack(int[] nums,int start,int end){
        if(start > end)return null;//左闭右闭
        
        int mid = start + (end - start) / 2;//不用mid = (left + right) / 2防止溢出
        TreeNode root = new TreeNode(nums[mid]);
        
        root.left = backtrack(nums,start,mid-1);
        root.right = backtrack(nums,mid+1,end);
            
        return root;
    }
}
```

#### 125.验证回文串

```java
class Solution {
    public boolean isPalindrome(String s) {
        int i = 0, j = s.length() - 1;
        while(i < j){
            while(i < j && !Character.isLetterOrDigit(s.charAt(i))) i++;
            while(i < j && !Character.isLetterOrDigit(s.charAt(j))) j--;
            if(Character.toLowerCase(s.charAt(i)) != Character.toLowerCase(s.charAt(j))) return false;
            i++; j--;
        }
        return true;
    }
}
```

#### 162.寻找峰值

```java
class Solution {
    public int findPeakElement(int[] nums) {
        int left = 0,right = nums.length-1;//因为一定会有坡顶上面-1，底下是<不是<=
        
        while(left < right){
            int mid = left + (right - left) / 2;
            
            if(nums[mid] > nums[mid+1])
                right = mid;//[left,mid]一定有坡顶
            else 
                left = mid + 1;//[mid+1,right)一定有坡顶
        }
        
        return left;//left = right返回谁都行
    }
}
```

#### 134.加油站

```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int sum = 0,cur = 0,res = 0;
        for(int i = 0; i < gas.length; i++){
            sum += gas[i] - cost[i];
            cur += gas[i] - cost[i];
            
            if(cur < 0){
                res = i+1;
                cur = 0;
            }
        }
        if(sum < 0)return -1;
        return res;
    }
}
```

#### 172.阶乘后的零

```java
class Solution {
    public int trailingZeroes(int n) {
        int count = 0;
        
        while(n >= 5){
            count += n/5;
            n /= 5;
        }
        return count;
    }
}
```

#### 179.最大数

```java
class Solution {
    public String largestNumber(int[] nums) {
        
        String[] s = new String[nums.length];
        for(int i = 0; i < nums.length; i++)
            s[i] = String.valueOf(nums[i]);
        
        Arrays.sort(s,(a,b)->(b+a).compareTo((a+b)));
        if(s[0].equals("0"))return "0";
        StringBuilder builder = new StringBuilder();
        for(int i = 0; i < nums.length; i++)
            builder.append(s[i]);
                     
        return builder.toString();
    }
}
```

#### 190.颠倒的二进制位

```java
public int reverseBits1(int n) {
    int result = 0;
    for (int i = 0; i <= 32; i++) {
        // 1. 将给定的二进制数,由低到高位逐个取出
        // 1.1 右移 i 位,
        int tmp = n >> i;
        // 1.2  取有效位
        tmp = tmp & 1;
        // 2. 然后通过位运算将其放置到反转后的位置.
        tmp = tmp << (31 - i);
        // 3. 将上述结果再次通过运算结合到一起
        result |= tmp;
    }
    return result;
}
```

#### 191.位1的个数

```java
public class Solution {
    // you need to treat n as an unsigned value
    public int hammingWeight(int n) {
        int mask = 1,count = 0;
        for(int i = 0; i < 32; i++){
            if((n & mask) != 0){
                //System.out.print((n & mask) +" ");
                count ++;
            }
            mask <<= 1;
        }
        //return Integer.bitCount(n);
        return count;
    }
}
```

#### 204.计数质数

```java
class Solution {
    public int countPrimes(int n) {
        boolean[] isPrim = new boolean[n];
        Arrays.fill(isPrim,true);
        for(int i = 2; i * i < n; i++){
            if(isPrim[i]){
                for(int j = 2* i; j < n; j += i){
                    isPrim[j] = false;
                }
            }
        }
        int count = 0;
        for(int i = 2; i < n ; i++){
            if(isPrim[i])
                count ++;
        }
        
        return count;
    }
}
```

#### 208.实现Trie(前缀树)

```java
class TreeNode{
    private TreeNode[] list;
    private boolean isEnd;
    
    public TreeNode getNode(char ch){
        return list[ch - 'a'];
    }
    public void setNode(char ch,TreeNode t){
        list[ch - 'a'] = t;
    }
    public boolean containsNode(char ch){
        return list[ch - 'a'] != null;
    }
    public void setEnd(){
        this.isEnd = true;
    }
    public boolean getEnd(){
        return isEnd;
    }
    public TreeNode() {
        list = new TreeNode[26];
    }

}
class Trie {
    public TreeNode root;
    /** Initialize your data structure here. */
    public Trie() {
        root = new TreeNode();
    }
    
    public TreeNode searchPrefix(String word){
        TreeNode node = root;
        for(int i = 0; i < word.length(); i++){
            char ch = word.charAt(i);
            
            if(node.containsNode(ch))
                node = node.getNode(ch);
            else
                return null;
        }
        return node;
    }
    
    /** Inserts a word into the trie. */
    public void insert(String word) {
        TreeNode node = root;
        for(int i = 0; i < word.length(); i++){
            char ch = word.charAt(i);
            
            if(!node.containsNode(ch))
                node.setNode(ch,new TreeNode());
            node = node.getNode(ch);
        }
        node.setEnd();
    }
    
    /** Returns if the word is in the trie. */
    public boolean search(String word) {
        TreeNode node = searchPrefix(word);
        return node != null && node.getEnd() == true;
    }
    
    /** Returns if there is any word in the trie that starts with the given prefix. */
    public boolean startsWith(String prefix) {
        return searchPrefix(prefix) != null;
    }
}

/**
 * Your Trie object will be instantiated and called as such:
 * Trie obj = new Trie();
 * obj.insert(word);
 * boolean param_2 = obj.search(word);
 * boolean param_3 = obj.startsWith(prefix);
 */
```

#### 289.生命游戏

```java
class Solution {
    public int mark(int[][] board,int in,int jn){
        int top = Math.max(in-1,0);
        int bottom = Math.min(in+1,board.length-1);
        int left = Math.max(jn-1,0);
        int right = Math.min(jn+1,board[0].length-1);
        int count = 0;
        for(int i = top; i <= bottom; i++){
            for(int j = left; j <= right; j++){
                
                if(board[i][j] == 1 || board[i][j] == -2)
                    count++;
            }
        }
        
        return board[in][jn] == 1? (count == 3 || count == 4 ? 1:-2) : (count == 3? -1:0);
    }
    public void gameOfLife(int[][] board) {
        //1 保持1;0 保持0;-1 0转1;-2 1转0
        if(board.length == 0)return;
        for(int i = 0; i < board.length; i++){
            for(int j = 0; j < board[0].length; j++){
                
                board[i][j] = mark(board,i,j);
            }
        }
        
        for(int i = 0; i < board.length; i++){
            for(int j = 0; j < board[0].length; j++){
                if(board[i][j] == 1 || board[i][j] == -1)
                    board[i][j] = 1;
                else
                    board[i][j] = 0;
            }
        }
    }
}
```

#### 324.摆动序列II

```java
class Solution {
    public void swap(int[] nums,int left,int right){
        int mid = (left + right)/2;
        for(int i = 0; i <= mid-left; i++){
            int tmp = nums[left + i];
            nums[left + i] = nums[right - i];
            nums[right - i] = tmp;
        }
    }
    public void wiggleSort(int[] nums) {
        if(nums.length == 0 || nums.length == 1)return;
        int mid = nums.length/2;
        Arrays.sort(nums);
        swap(nums,0,mid-1);
        swap(nums,mid,nums.length-1);
        int j = 0;
        int[] res = new int[nums.length];
        if(nums.length%2 == 1){
            res[j++] = nums[nums.length-1];
            for(int i = 0; i < mid; i++){
                res[j++] = nums[mid+i];
                res[j++] = nums[i];
            }
        }else{
            for(int i = 0; i < mid; i++){
                res[j++] = nums[i];
                res[j++] = nums[mid+i];
            }
        }
        System.arraycopy(res, 0, nums, 0, nums.length);
    }
}
```

#### 334.递增的三元子序列

```java
class Solution {
    public boolean increasingTriplet(int[] nums) {
        
        int one = Integer.MAX_VALUE;
        int two = Integer.MAX_VALUE;
        
        for(int num : nums){
            if(num <= one)
                one = num;
            else if(num <= two)
                two = num;
            else
                return true;
        }
        
        return false;
    }
}
```

#### 371.两整数之和

```java
class Solution {
    public int getSum(int a, int b) 
    {
        int sum, carry; 
        sum = a ^ b;  
            //异或这里可看做是相加但是不显现进位，比如5 ^ 3
                 /*0 1 0 1
                   0 0 1 1
                 ------------
                   0 1 1 0      
              上面的如果看成传统的加法，不就是1+1=2，进1得0，但是这里没有显示进位出来，仅是相加，0+1或者是1+0都不用进位*/
    
        carry = (a & b) << 1;
    
                //相与为了让进位显现出来，比如5 & 3
                /* 0 1 0 1
                   0 0 1 1
                 ------------
                   0 0 0 1
              上面的最低位1和1相与得1，而在二进制加法中，这里1+1也应该是要进位的，所以刚好吻合，但是这个进位1应该要再往前一位，所以左移一位*/
    
        if(carry != 0)  //经过上面这两步，如果进位不等于0，那么就是说还要把进位给加上去，所以用了尾递归，一直递归到进位是0。
            return getSum(sum, carry);
        return sum;
    }
}
```

#### 378.有序矩阵中第K小的元素

```java
class Solution {
        public int kthSmallest(int[][] matrix, int k) {
        int row = matrix.length;
        int col = matrix[0].length;
        int left = matrix[0][0];
        int right = matrix[row - 1][col - 1];
        while (left < right) {
            // 每次循环都保证第K小的数在start~end之间，当start==end，第k小的数就是start
            int mid = (left + right) / 2;
            // 找二维矩阵中<=mid的元素总个数
            int count = findNotBiggerThanMid(matrix, mid, row, col);
            if (count < k) {
                // 第k小的数在右半部分，且不包含mid
                left = mid + 1;
            } else {
                // 第k小的数在左半部分，可能包含mid
                right = mid;
            }
        }
        return right;
    }

    private int findNotBiggerThanMid(int[][] matrix, int mid, int row, int col) {
        // 以列为单位找，找到每一列最后一个<=mid的数即知道每一列有多少个数<=mid
        int i = row - 1;
        int j = 0;
        int count = 0;
        while (i >= 0 && j < col) {
            if (matrix[i][j] <= mid) {
                // 第j列有i+1个元素<=mid
                count += i + 1;
                j++;
            } else {
                // 第j列目前的数大于mid，需要继续在当前列往上找
                i--;
            }
        }
        return count;
    }
}
```

#### 384.打乱数组

```java
class Solution {
    int[] arr;
    int[] copy;
    public void swap(int i,int j){
        int tmp = arr[i];//这里不能用异或运算，因为可能是两个相同的数字异或为0了。
        arr[i] = arr[j];
        arr[j] = tmp;
    }
    public Solution(int[] nums) {
        arr = nums;
        copy = nums.clone();
    }
    
    /** Resets the array to its original configuration and return it. */
    public int[] reset() {
        return arr = copy.clone();
    }
    
    /** Returns a random shuffling of the array. */
    public int[] shuffle() {
        for(int i = 0; i < arr.length; i++){
            Random rm = new Random();
            int j = rm.nextInt(arr.length-i) + i;
            swap(i,j);
        }
        
        return arr;
    }
}

/**
 * Your Solution object will be instantiated and called as such:
 * Solution obj = new Solution(nums);
 * int[] param_1 = obj.reset();
 * int[] param_2 = obj.shuffle();
 */
```

#### 454.四数相加

```java
class Solution {
    public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
        Map<Integer,Integer> map = new HashMap<>();
        
        for(int i = 0; i < A.length; i++){
            for(int j = 0; j < B.length; j++){
                int sum = A[i] + B[j];
                map.put(sum,map.getOrDefault(sum,0)+1);
            }
        }
        
        int count = 0;
        for(int i = 0;i < C.length; i++){
            for(int j = 0;j < D.length; j++){
                int sum = C[i] + D[j];
                sum *= -1;
                if(map.containsKey(sum)){
                    count += map.get(sum);
                }
            }
        }
        
        return count;
    }
}
```