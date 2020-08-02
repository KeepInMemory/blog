---
title: CyC-LeetCode二刷记录
date: 2020-06-20 19:55:53
tags:
  - LeetCode
---

### 双指针

633.平方数之和：a和b是可以相等的，所以while(i < j)改成while(i <= j)

345.反转字符串的元音字母：想要改String的某个位置的字符

```java
char[] c = s.toCharArray();//速度快
c[i] = tmp;

StringBuilder sb = new StringBuilder(s);//速度慢
sb.setCharAt(i,tmp);
```

88.合并俩个数组：不熟练

524.通过删除字母匹配到字典里最长单词

长度优先，如果长度相同的字典序小的优先

String字典序通过compareTo比较，判断是否是字串通过双指针

```java
class Solution {
    public boolean isSubStr(String a,String b){
        int i = 0,j = 0;
        while(i < a.length() && j < b.length()){
            if(a.charAt(i) == b.charAt(j)) 
                j++;
            i++;
        }
        return j == b.length();
    }
    public String findLongestWord(String s, List<String> d) {
        String LongestString = "";

        for(int i = 0; i < d.size(); i ++){
            String cur_str = d.get(i);
            int len1 = LongestString.length(),len2 = cur_str.length();
            if(len1 > len2 || (len1 == len2&&(cur_str.compareTo(LongestString) >= 0))) continue;

            if(isSubStr(s,cur_str)) {
                LongestString = cur_str;
            }
        }
        return LongestString;
    }
}
```

### 排序

#### 堆|快排

215.数组中的第K个最大元素

堆排序，模仿维护堆的出入过程

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        PriorityQueue<Integer> pq = new PriorityQueue<>();

        for(int num:nums) {
            pq.add(num);
            if(pq.size() > k) {
                pq.poll();
            }
        }
        return pq.peek();
    }
}
```

快排

```java
class Solution {
    public void swap(int[] nums,int i,int j){
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }

    public int partition(int[] nums,int left,int right) {
        Random r = new Random();
        int index = r.nextInt(right-left+1) + left;
        int pivot = nums[index];
        swap(nums,left,index);
        int i = left+1,j = right;
        while(true){
            while(i < right && nums[i] <= pivot){i++;}
            while(j > left && nums[j] >= pivot){j--;}
            if(i >= j)break;
            swap(nums,i,j);
        }
        swap(nums,left,j);//j的位置是对的，i不对，因为i前面多了一个在nums[left]上的pivot，而j是从后面数的，没有这个，所以位置是准确的
        return j;
    }

    public int findKthLargest(int[] nums, int k) {
        int left = 0,right = nums.length-1;
        int target = nums.length-k;
        int res = 0;
        while(true) {
            res = partition(nums,left,right);
            if(res == target)break;
            else if(res < target) {
                left = res + 1;
            }else {
                right = res - 1;
            }
        }
        return nums[res];
    }
}
```

#### 桶排序

347.前 K 个高频元素

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        //哈希表计数
        HashMap<Integer,Integer> map = new HashMap<>();
        for(int num: nums) {
            //map.put(num,map.getOrDefault(num,0)+1);这样比较慢
            if(map.containsKey(num))
                map.put(num,map.get(num)+1);
            else
                map.put(num,1);
        }
        //桶排序
        List<Integer>[] bucket = new List[nums.length+1];
        for(int num:map.keySet()) {
            int frequency = map.get(num);
            if(bucket[frequency] == null)
                bucket[frequency] = new ArrayList<Integer>();
            bucket[frequency].add(num);
        }
        //准备最后结果
        int[] res = new int[k];
        int count = 0;
        for(int i = bucket.length-1; i >= 1; i--){
            if(bucket[i] != null) {
                for(int num:bucket[i]) {
                    if(count >= k) break;
                    res[count++] = num;
                }
            }
        }
        return res;
    }
}
```

451.根据字符出现频率排序

```java
class Solution {
    public String frequencySort(String s) {
        //哈希表计数
        HashMap<Character,Integer> map = new HashMap<>();
        for(int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if(map.containsKey(c))
                map.put(c,map.get(c)+1);
            else
                map.put(c,1);
        }

        //桶排序
        List<Character>[] bucket = new List[s.length()+1];
        for(Character c:map.keySet()) {
            int frequency = map.get(c);
            if(bucket[frequency] == null)
                bucket[frequency] = new ArrayList<>();
            bucket[frequency].add(c);
        }

        //构造结果
        StringBuffer sb = new StringBuffer();
        for(int i = bucket.length-1; i >= 1; i--) {
            if(bucket[i] != null) {
                for(Character c:bucket[i]) {
                    for(int j = 0; j < i; j++)
                        sb.append(c);
                }
            }
        }
        return sb.toString();
    }
}
```

尝试用堆排序

```java
class Solution {
    public String frequencySort(String s) {
        //哈希表计数
        HashMap<Character,Integer> map = new HashMap<>();
        for(int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if(map.containsKey(c))
                map.put(c,map.get(c)+1);
            else
                map.put(c,1);
        }

        //堆排序
        PriorityQueue<Character> pq = new PriorityQueue<>((o1,o2) -> (map.get(o2)-map.get(o1)));//大顶堆
        pq.addAll(map.keySet());//注意这里的使用方式
        StringBuffer sb = new StringBuffer();
        while(!pq.isEmpty()) {
            char c = pq.poll();
            for(int i = 0; i < map.get(c); i++)
                sb.append(c);
        }
        return sb.toString();
    }
}
```

addAll(Collection<E>) 而map.keySet()返回key的Collection<E>

这里重写的compare，默认是o1-o2，从小到大。如果要从大到小就需要反着o2-o1

75.颜色分类

```java
class Solution {
    public void swap(int[] nums,int i,int j) {
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
    public void sortColors(int[] nums) {
        //三指针
        int zero = -1,one = 0,two = nums.length;
        while(one < two) {
            if(nums[one] == 0) swap(nums,++zero,one++);
            else if(nums[one] == 2) swap(nums,one,--two);
            else one++;
        }
    }
}
```

三指针，将遇到的0放到首部，2放到尾部，zero标识首部0的最后，two标识尾部2的开头

注意不能让zero = 0,one = 0,two = nums.length-1

### 贪心

455.分发饼干

```java
class Solution {
    public int findContentChildren(int[] g, int[] s) {
        //贪心
        int i = 0,j = 0;
        Arrays.sort(g);
        Arrays.sort(s);
        while(i < g.length && j < s.length) {
            if(g[i] <= s[j])
                i++;
            j++;
        }
        return i;
    }
}
```

435.无重叠区间

```java
class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        if(intervals.length == 0) return 0;
        Arrays.sort(intervals,new Comparator<int[]>() {
            public int compare(int[] o1,int[] o2) {
                return o1[1] - o2[1];
            }
        });

        int count = 1;
        int end = intervals[0][1];
        for(int i = 1; i < intervals.length; i++) {
            if(end <= intervals[i][0]) {
                end = intervals[i][1];
                count ++;
            }
        }
        return intervals.length - count;
    }
}
```

452.用最少数量的箭引爆气球

```java
class Solution {
    public int findMinArrowShots(int[][] points) {
        //计算重叠区间个数
        if(points.length == 0)return 0;
        Arrays.sort(points,new Comparator<int[]>() {
            public int compare(int[] o1,int[] o2) {
                return o1[1] - o2[1];
            }
        });

        int count = 1;
        int end = points[0][1];
        for(int i = 1; i < points.length; i++) {
            if(end < points[i][0]) {
                count ++;
                end = points[i][1];
            }
        }
        return count;
    }
}
```

406.根据身高重建队列

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        //先按身高降序，再按序号升序，插入序号位置，如果有人了则依次往后排，对其他人的顺序不影响
        Arrays.sort(people, new Comparator<int[]>() {
            public int compare(int[] o1,int[] o2) {
                return o1[0] == o2[0]? o1[1] - o2[1]:o2[0] - o1[0];
            }
        });
        List<int[]> list = new ArrayList();
        for(int[] p : people) {
            list.add(p[1],p);
        }
        return list.toArray(new int[people.length][2]);
    }
}
```

121.买卖股票的最佳时机

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices.length == 0)return 0;
        int min = prices[0],max = 0;
        for(int p : prices) {
            if(min > p) min = p;//不会出现前面的减去后面的min
            else max = Math.max(max,p-min);
        }
        return max;
    }
}
```

122.买卖股票的最佳时机Ⅱ

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices.length == 0)return 0;
        int profit = 0;
        for(int i = 1;i < prices.length; i++) {
            if(prices[i] > prices[i-1])
                profit += prices[i] - prices[i-1];
        }
        return profit;
    }
}
```

`[7, 1, 5, 6]`  第二天买入，第四天卖出，收益最大（6-1），所以一般人可能会想，怎么判断不是第三天就卖出了呢?  这里就把问题复杂化了，根据题目的意思，当天卖出以后，当天还可以买入，所以其实可以第三天卖出，第三天买入，第四天又卖出（（5-1）+ （6-5）  === 6 - 1）。所以算法可以直接简化为只要今天比昨天大，就卖出。

605.种花问题

```java
class Solution {
    public boolean canPlaceFlowers(int[] flowerbed, int n) {
        int count = 0;
        int i = 0;
        while(i < flowerbed.length) {
            if(flowerbed[i] == 0) {
                int pre = i==0?0:flowerbed[i-1];
                int next = i==flowerbed.length-1?0:flowerbed[i+1];
                if(pre == next && pre == 0) {
                    count ++;
                    flowerbed[i] = 1;//关键
                }
            }
            i++;
        }
        return count >= n;
    }
}
```

392.判断子序列

用双指针是可以实现的，但CyC的实现更好

使用了String的indexOf(c,i)，标识从i位置开始找c，找到返回下标没找到返回-1

665.非递减数列

```java
class Solution {
    public boolean checkPossibility(int[] nums) {
        //因为是递增序列，所以数字变大是无所谓的，只要发现数字变小了就不行
        int count = 1;
        for(int i = 0; i < nums.length-1 ; i++) {
            if(nums[i] > nums[i+1]) {
                if(count-- == 0)return false;
                if(i == 0 || nums[i-1] <= nums[i+1])
                    nums[i] = nums[i+1];
                else if(nums[i-1] > nums[i+1])
                    nums[i+1] = nums[i];
            }
        }
        return true;
    }
}
```

当 i 和 i+1 构成逆序时，要么修改nums[i]要么修改nums[i+1]，即要么i大了 i+1正常，要么i+1小了 i正常

- 如果 i-1 和 i+1 是降序排列，说明nums[i+1]小了，此时增大 i+1 的值
- 如果 i-1 和 i+1 是升序排列，说明nums[i]大了，此时缩小 i 的值

53.最大子序和

假设sum<=0，那么后面的子序列肯定不包含目前的子序列，所以令sum = num；如果sum > 0对于后面的子序列是有好处的。res = Math.max(res, sum)保证可以找到最大的子序和。

763.划分字母区间

```java
class Solution {
    public List<Integer> partitionLabels(String S) {
        if(S == null || S.length() == 0)return new ArrayList<Integer>();
        int[] index = new int[26];
        for(int i = 0; i < S.length(); i++) {
            index[S.charAt(i) -'a'] = i;
        }
        
        List<Integer> res = new ArrayList<>();
        int left = 0,right = index[S.charAt(0)-'a'];
        for(int i = 0; i < S.length(); i++) {
            right = Math.max(right,index[S.charAt(i)-'a']);
            if(i == right) {
                res.add(i-left+1);
                left = right+1;
            }
        }
        return res;
    }
}
```

### 二分查找

69.x的平方根

```java
class Solution {
    public int mySqrt(int x) {
        if(x <= 1)return x;
        long left = 1, right = x;
        while(left < right) {
            long mid = left+(right-left+1)/2;
            long mpl = mid * mid;
            if(mpl > x) right = mid - 1;
            else left = mid;
        }
        return (int)right;
    }
}
```

mid 和 mpl容易越界，所以改成long

mid = left+(right-left+1)是上取整

mid=left+(right-left)是下取整

因为left没有等于mid+1，所以有可能会出现left=2,mid=2,right=3，而2的位置又正好mpl<=x，所以会出现死循环，所以每次都要上取整，left=2,mid=3,right=3，尽量让他right=mid-1，而不是left=mid

总之就是上取整只会出现在这种情况

540.有序数组中的单一元素

根据索引判断，将索引分为奇数偶数，奇数的话该索引和它前一个位置应该是相同的元素，如果不相同说明单个元素在后面，这里的right不能等于mid+1，因为mid也是有可能的，而奇数下的mid因为已经有相同的了所以直接跳过left=mid+1。偶数的地方同理。

### 分治

241.为运算表达式设计优先级

```java
class Solution {
    public List<Integer> diffWaysToCompute(String input) {
        List<Integer> res = new ArrayList<>();

        for(int i = 0; i < input.length(); i++) {
            char c = input.charAt(i);
            if(c == '+' || c == '-' || c == '*'){
                //左边所有可能的结果
                List<Integer> left = diffWaysToCompute(input.substring(0,i));
                //右边所有可能的结果
                List<Integer> right = diffWaysToCompute(input.substring(i+1));

                for(int l : left) {
                    for(int r : right) {
                        switch(c) {
                            case '+' : res.add(l + r);break;
                            case '-' : res.add(l - r);break;
                            case '*' : res.add(l * r);break;
                        }
                    }
                }
            }

        }
        //左或右或者全部没有运算符，只有数字的情况
        if(res.size() == 0) {
            res.add(Integer.valueOf(input));
        }

        return res;
    }
}
```

95.搜索不同的二叉树Ⅱ

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<TreeNode> generateTrees(int n) {
        List<TreeNode> res = new LinkedList<>();
        if(n < 1) return res;
        res = generateSubTrees(1,n);
        return res;
    }
    public List<TreeNode> generateSubTrees(int start,int end) {
        List<TreeNode> list = new LinkedList<>();
        if(start > end) 
            list.add(null);
        for(int i = start; i <= end; i++) {
            List<TreeNode> left = generateSubTrees(start,i-1);
            List<TreeNode> right = generateSubTrees(i+1,end);

            for(TreeNode l:left) {
                for(TreeNode r:right) {
                    TreeNode node = new TreeNode(i,l,r);
                    list.add(node);
                }
            }
        }
        return list;
    }
}
```

### 搜索

#### BFS

求解图的最短路径最优解

127.单词接龙

```java
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        int step = 1;
        Queue<String> queue = new LinkedList<>();
        Set<String> words = new HashSet<>(wordList);
        Set<String> visited = new HashSet<>();
        if(words.size() == 0 || !words.contains(endWord)) return 0;
        queue.add(beginWord);
        visited.add(beginWord);
        words.remove(beginWord);

        while(!queue.isEmpty()) {
            
            int queueSize = queue.size();
            for(int j = 0; j < queueSize; j++) {
                String cur = queue.poll();
                char[] ch = cur.toCharArray();
                int wordLen = beginWord.length();
                for(int k = 0; k < wordLen; k++) {
                    char originChar = ch[k];

                    for(char h = 'a'; h <= 'z'; h++) {
                        if(h == originChar) continue;
                        ch[k] = h;
                        String s = String.valueOf(ch);
                        if(words.contains(s)) {
                            if(s.equals(endWord)) return step+1;
                            
                            if(!visited.contains(s)) {
                                queue.add(s);
                                visited.add(s);
                            }
                        }
                    }
                    ch[k] = originChar;
                }
            }
            step ++;
        }
        return 0;
    }
}
```

#### DFS

求解可达性问题

695.岛屿的最大面积

```java
class Solution {
    public int maxAreaOfIsland(int[][] grid) {
        int max = 0;
        int row = grid.length,col = grid[0].length;
        for(int i = 0; i < row; i++) {
            for(int j = 0; j < col; j++) {
                if(grid[i][j] == 1) {
                    max = Math.max(dfs(grid,i,j),max);
                }
            }
        }
        return max;
    }
    public int dfs(int[][] grid,int i,int j) {
        if(i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] == 0)
            return 0;
        
        int area = 1;
        grid[i][j] = 0;
        area += dfs(grid,i+1,j);
        area += dfs(grid,i-1,j);
        area += dfs(grid,i,j+1);
        area += dfs(grid,i,j-1);
        return area;
    }
}
```

200.岛屿数量

```java
class Solution {
    public int numIslands(char[][] grid) {
        if(grid == null || grid.length == 0)return 0;
        int sum = 0;
        int row = grid.length,col = grid[0].length;
        for(int i = 0; i < row; i++) {
            for(int j = 0; j < col; j++) {
                if(grid[i][j] == '1') {
                    sum ++;
                    dfs(grid,i,j);
                }
            }
        }
        return sum;
    }

    public void dfs(char[][] grid,int i,int j) {
        if(i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] == '0')
            return;
        
        int sum = 1;
        grid[i][j] = '0';

        dfs(grid,i+1,j);
        dfs(grid,i-1,j);
        dfs(grid,i,j+1);
        dfs(grid,i,j-1);
    }
}
```

dfs的功能是把从一个陆地开始能遍历的所有陆地走一遍，走过的变成海洋。计过数的岛屿不会再计因为都变成了海洋。

547.朋友圈

```java
class Solution {
    public int findCircleNum(int[][] M) {
        if(M == null || M.length == 0)return 0;
        boolean[] visited = new boolean[M.length];
        int res = 0;
        for(int i = 0; i < M.length; i++) {
            if(!visited[i]) {
                dfs(M,visited,i);
                res ++;
            }
        }
        return res;
    }
    public void dfs(int[][] M,boolean[] visited,int i) {
        for(int j = 0; j < M.length; j++) {
            if(M[i][j] == 1 && !visited[j]) {
                visited[j] = true;
                dfs(M,visited,j);
            }
        }
    }
}
```

visited表示已经记过次数的朋友圈，没有visit说明外面还有朋友圈，就dfs查询所有朋友全部加入visited

130.被围绕的区域

```java
class Solution {
    public void solve(char[][] board) {
        if(board == null || board.length == 0)return;
        int row = board.length,col = board[0].length;
        for(int i = 0; i < row; i++) {
            for(int j = 0; j < col; j++) {
                if((i == 0 || j == 0 || i == row-1 || j == col-1) && board[i][j] == 'O') {
                    dfs(board,i,j);
                }
            }
        }
        for(int i = 0; i < row; i++) {
            for(int j = 0; j < col; j++) {
                if(board[i][j] == '#') board[i][j] = 'O';
                else if(board[i][j] == 'O') board[i][j] = 'X';
            }
        }
    }

    public void dfs(char[][] board,int i,int j) {
        if(i < 0 || i  >= board.length || j < 0 || j >= board[0].length || board[i][j] == 'X' || board[i][j] == '#') return;

        board[i][j] = '#';
        dfs(board,i-1,j);
        dfs(board,i+1,j);
        dfs(board,i,j-1);
        dfs(board,i,j+1);
    }
}
```

从边界的O开始，遍历所有能达到的O区域，全部置为#。最后全部遍历，#置为O，第一次没遍历到的O变为X

417.太平洋大西洋水流问题

一个一个元素dfs会超时，从四条边的元素开始dfs，意思是能达到四条边的所有区域都标记出来，找两个数组重合的地方就是能同时达到两个大洋的地方

#### 回溯

17.电话号码的字母组合

```java
class Solution {
    public List<String> letterCombinations(String digits) {
        if(digits.length() == 0) return new LinkedList<>();
        String[] strs = {"","","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
        List<String> res = new LinkedList<>();
        combine(digits,strs,0,"",res); 
        return res;
    }

    public void combine(String digits,String[] strs,int i,String cur,List<String> res) {
        if(i == digits.length()) {
            res.add(cur);
            return;
        }
        int index = digits.charAt(i)-'0';
        String str = strs[index];
        for(int j = 0; j < str.length(); j++) {
            combine(digits,strs,i+1,cur+str.charAt(j),res);
        }
    }
}
```

93.复原IP地址

```java
class Solution {
    public List<String> restoreIpAddresses(String s) {
        if(s == null || s.length() == 0 || s.length() > 12) return new ArrayList<>();
        List<String> res = new LinkedList<>();
        StringBuilder sb = new StringBuilder();
        backtracking(s,0,0,sb,res);
        return res;
    }

    public void backtracking(String s,int count,int index,StringBuilder sb,List<String> res) {
        int len = sb.length();
        if(count == 4) {
            if(len == s.length() + 4) {
                sb.deleteCharAt(len-1);
                res.add(sb.toString());
            }
        }

        for(int i = 0; i < 3; i++) {
            if(index + i >= s.length()) break;//会出现剩下一个数字，但是i=2了，底下取子串会越界
            String sub = s.substring(index,index+i+1);
            int num = Integer.valueOf(sub);
            if(num > 255) break;//判断是否大于255
            String str = String.valueOf(num);
            if(str.length() != sub.length()) break;//判断01.001这些情况

            sb.append(sub + ".");
            backtracking(s,count+1,index+i+1,sb,res);
            sb.delete(len,len + sub.length() + 1);
        }
    }
}
```

79.在矩阵中寻找字符串 不熟练

257.二叉树的所有路径 不熟练

46.全排列

for从0开始

47.全排列Ⅱ

for从0开始，有boolean[] visited。if(i > 0 && nums[i] == nums[i-1] && !visited[i-1]) continue; 表示如果这次的数和它前面的数是一样的，而且visited[i-1]=false前面那个相同的数还没被访问过，那么必定一会构造的排列会有前面那个相同的数，一定会产生相同的排列，因为这次排列前面的排列是从前往后排的，一定是前面的先排再是后面的排，如果前面的那个相同数还没排说明一定有重复的排列了，应该舍弃

77.组合

从Start开始，不是从0开始。回溯法解决组合问题。和排序问题不同的是，在组合问题中元素的顺序不考虑，只需要从当前位置向后寻找。排序问题每次都需要从头寻找，需要用visited数组记录访问过的元素。

39.组合总和

注意也是从Start开始，不是从0开始，因为是初始数组是有序的

40.组合总和Ⅱ

从Start开始，不是从0开始，有boolean[] visited。排序后用if(i > 0 && nums[i] == nums[i-1] && !visited[i-1]) continue;避免重复结果

216.组合总和Ⅲ

78.子集

从start开始

90.子集Ⅱ

数组排序，从start开始，有boolean[] visited。if(i > 0 && nums[i] == nums[i-1] && !visited[i-1]) continue; 

131.分割回文串

### 动态规划

70.爬楼梯

198.打家劫舍

表达式会写

213.打家劫舍Ⅱ

把环拆成两个队列，一个是从0到n-1，另一个是从1到n，然后返回两个结果最大的。

64.最小路径和

注意表达式

62.不同路径

我们令 dp[i][j] 是到达 i, j 最多路径

动态方程：dp[i][j] = dp[i-1][j] + dp[i][j-1]

注意，对于第一行 dp[0][j]，或者第一列 dp[i][0]，由于都是在边界，所以只能为 1（横着或者竖着走，都是一条路径）

413.等差数列划分

```java
dp[2] = 1
    [0, 1, 2]
dp[3] = dp[2] + 1 = 2
    [0, 1, 2, 3], // [0, 1, 2] 之后加一个 3
    [1, 2, 3]     // 新的递增子区间
dp[4] = dp[3] + 1 = 3
    [0, 1, 2, 3, 4], // [0, 1, 2, 3] 之后加一个 4
    [1, 2, 3, 4],    // [1, 2, 3] 之后加一个 4
    [2, 3, 4]        // 新的递增子区间
```

综上，在 A[i] - A[i-1] == A[i-1] - A[i-2] 时，dp[i] = dp[i-1] + 1。

因为递增子区间不一定以最后一个元素为结尾，可以是任意一个元素结尾，因此需要返回 dp 数组累加的结果。

343.整数拆分

```java
class Solution {
    public int IntegerBreak(int n) {
        int[] dp = new int[n+1]; 
        if(n <= 3) return n-1;
        /*解决大问题的时候用到小问题的解并不是这三个数
        真正的dp[1] = 0,dp[2] = 1,dp[3] = 2
        但是当n=4时，4=2+2 2*2=4 而dp[2]=1是不对的
        也就是说当n=1/2/3时，分割后反而比没分割的值要小，当大问题用到dp[j]时，说明已经分成了一个j一个i-j，这两部分又可以再分，但是再分不能比他本身没分割的要小，如果分了更小还不如不分
        所以在这里指定大问题用到的dp[1],dp[2],dp[3]是他本身*/
        dp[1] = 1;dp[2] = 2;dp[3] = 3;
        for(int i = 4; i <= n; i++) {
            for(int j = 1; j <= i/2; j++) {
                dp[i] = Math.max(dp[i],dp[i-j]*dp[j]);
            }
        }
        return dp[n];
    }
}
```

300.最长上升子序列

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        if(nums.length == 0)return 0;
        int[] dp = new int[nums.length];
        int max = 1;
        for(int i = 0; i < nums.length; i++) {
            for(int j = 0; j < i; j++) {
                if(nums[i] > nums[j]) {
                    dp[i] = Math.max(dp[i],dp[j]+1);
                }
            }
            max = Math.max(max,dp[i]+1);
        }

        return max;
    }

```

646.最长数对链

```java
class Solution {
    public int findLongestChain(int[][] pairs) {
        Arrays.sort(pairs,new Comparator<>() {
            public int compare(int[] p1,int[] p2) {
                return p1[1] - p2[1];
            }
        });

        if(pairs.length == 0) return 0;
        int[] dp = new int[pairs.length];

        int max = 1;
        for(int i = 0; i < pairs.length; i++) {
            for(int j = 0; j < i; j++) {
                if(pairs[j][1] < pairs[i][0]) {
                    dp[i] = Math.max(dp[i],dp[j]+1);
                }
            }
            max = Math.max(dp[i]+1,max);
        }
        return max;
    }
}
```

