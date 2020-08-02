---
title: 剑指Offer刷题记录
date: 2019-11-19 12:00:00
tags:
  - 剑指Offer
---

#### 替换空格

```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
        String s = str.toString();
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < s.length(); i++){
            if(s.charAt(i) == ' ')
                builder.append("%20");
            else
                builder.append(s.charAt(i));
        }
        return builder.toString();
    }
}
```

```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
        return str.toString().replaceAll(" ","%20");
    }
}
```

#### 从头到尾打印链表

```java
import java.util.ArrayList;
public class Solution {
    ArrayList<Integer> list = new ArrayList<>();
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        if(listNode != null){
            printListFromTailToHead(listNode.next);
            list.add(listNode.val);
        }
        return list;
    }
}
```

#### 重建二叉树

```java
public class Solution {
    public TreeNode treeHelper(int[]pre,int ps,int pe,int[] in,int is,int ie) {
        if(ps > pe || is > ie)return null;
        TreeNode root = new TreeNode(pre[ps]);
        int i = 0;
        for(i = is; i <= ie; i++){
            if(in[i] == pre[ps])
                break;
        }
        int leftNum = i - is;
        int rightNum = ie - i;
        root.left = treeHelper(pre,ps+1,ps+leftNum,in,is,i-1);
        root.right = treeHelper(pre,ps+leftNum+1,ps+leftNum+rightNum,in,i+1,ie);
        
        return root;
    }
    public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
        return treeHelper(pre,0,pre.length-1,in,0,in.length-1);
    }
}
```

#### 用两个栈实现队列

```java
import java.util.Stack;

public class Solution {
    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();
    
    public void push(int node) {
        stack1.push(node);
    }
    
    public int pop() {
        while(!stack1.isEmpty())
            stack2.push(stack1.pop());
        int res = stack2.pop();
        while(!stack2.isEmpty())
            stack1.push(stack2.pop());
        
        return res;
    }
}
```

#### 旋转数组的最小数字

```java
import java.util.ArrayList;
public class Solution {
    public int minNumberInRotateArray(int [] array) {
        if(array.length == 0)return 0;
        int left = 0,right = array.length-1;
        while(left <= right){
            int mid = left + (right-left)/2;
            if(array[mid] > array[right])
                left = mid + 1;
            else if(array[mid] < array[right])
                right = mid;
            else
                return array[mid];
        }
        return 0;
    }
}
```

#### 斐波那契数列

```java
public class Solution {
    public int Fibonacci(int n) {
        if(n == 0)return 0;
        if(n == 1)return 1;
        int n1 = 0,n2 = 1;
        int res = 0;
        for(int i = 2; i <= n; i++){
            res = n1 + n2;
            n1 = n2;
            n2 = res;
        }
        return res;
    }
}
```

最好不要用递归

#### 跳台阶

```java
public class Solution {
    public int helper(int[] dp,int cur){
        if(cur == 1)return 1;
        if(cur == 2)return 2;
        return helper(dp,cur-1) + helper(dp,cur-2);
    }
    public int JumpFloor(int target) {
        int[] dp = new int[target+1];
        return helper(dp,target);
    }
}
```

跳台阶其实就是斐波那契额数列，如果想跳到6且只跳一步，从5跳到6跳一次一步，而且跳到5有多少种方案到6就有多少种，就是在最后再一个1，方案数目不变。从4跳到6跳一次两步，同理，4到6在所有方案最后加一个2，方案数目不变。4不能跳两个一步，因为违反了跳一次的规则，4跳一步的情况已经包含在了跳到5中。

#### 变态跳台阶

```java
public class Solution {
    public int JumpFloorII(int target) {
        if(target == 1)return 1;
        if(target == 2)return 2;
        int[] dp = new int[target+1];
        dp[0] = 0;dp[1] = 1;dp[2] = 2;
        for(int i = 3; i <= target; i ++){
            int res = 0;
            for(int j = 0; j < i; j ++){
                res += dp[j];
            }
            dp[i] = res + 1;
        }
        return dp[target];
    }
}
```

从1加到i-1后注意要加1，因为能一步跳上去

#### 矩阵覆盖

同样是斐波那契数列

#### 二进制中1的个数

```java
public class Solution {
    public int NumberOf1(int n) {
        int count = 0;
        
        while(n != 0){
            count ++;
            n = n&(n-1);
        }
        return count;
    }
}
```

 如果一个整数不为0，那么这个整数至少有一位是1。如果我们把这个整数减1，那么原来处在整数最右边的1就会变为0，原来在1后面的所有的0都会变成1(如果最右边的1后面还有0的话)。其余所有位将不会受到影响。
举个例子：一个二进制数1100，从右边数起第三位是处于最右边的一个1。减去1后，第三位变成0，它后面的两位0变成了1，而前面的1保持不变，因此得到的结果是1011.我们发现减1的结果是把最右边的一个1开始的所有位都取反了。这个时候如果我们再把原来的整数和减去1之后的结果做与运算，从原来整数最右边一个1那一位开始所有位都会变成0。如1100&1011=1000.也就是说，把一个整数减去1，再和原整数做与运算，会把该整数最右边一个1变成0.那么一个整数的二进制有多少个1，就可以进行多少次这样的操作。 

#### 反转链表

```java
public class Solution {
    public ListNode ReverseList(ListNode head) {
        if(head == null)return null;
        ListNode node = head,pre = null,next = head.next;
        while(next != null){
            node.next = pre;
            pre = node;
            node = next;
            next = next.next;
        }
        node.next = pre;
        return node;
    }
}
```

#### 树的子结构

```java
public class Solution {
    public boolean validate(TreeNode root1,TreeNode root2){
        if(root2 == null)return true;
        if(root1 == null)return false;
        if(root1.val != root2.val)return false;
        
        return validate(root1.left,root2.left) && validate(root1.right,root2.right);
    }
    public boolean HasSubtree(TreeNode root1,TreeNode root2) {
        if(root1 == null || root2 == null)return false;
        boolean result = false;
        if(root1.val == root2.val)  result = validate(root1,root2);
        if(result == false)result = HasSubtree(root1.left,root2) || HasSubtree(root1.right,root2);
        return result;
    }
}
```

#### 栈的压入、弹出序列

```java
import java.util.ArrayList;
import java.util.Stack;
public class Solution {
    public boolean IsPopOrder(int [] pushA,int [] popA) {
        Stack<Integer> stack = new Stack<>();
        int pop = 0;
        for(int push = 0; push < pushA.length; push++){
            stack.push(pushA[i]);
            
            while(!stack.isEmpty() && stack.peek() == popA[pop]){
                stack.pop();
                pop++;
            }
        }
        return stack.isEmpty();
    }
}
```

#### 连续子数组的最大和

```java
import java.lang.Math;
public class Solution {
    public int FindGreatestSumOfSubArray(int[] array) {
        int[] dp = new int[array.length+1];
        int res = array[0];
        for(int i = 1; i <= array.length; i++){
            dp[i] = Math.max(dp[i-1]+array[i-1],array[i-1]);
            res = Math.max(dp[i],res);
        }
        return res;
    }
}
```

#### 二叉搜索树的后序遍历序列

```java
public class Solution {
    public boolean helper(int[] sequence,int start,int end){
        if(start >= end)return true;
        int i = start;
        for(;i < end; i++){
            if(sequence[i] > sequence[end])
                break;
        }
        for(int j = i; j < end; j++){
            if(sequence[j] < sequence[end])
                return false;
        }
        
        return helper(sequence,start,i-1) && helper(sequence,i,end-1);
    }
    public boolean VerifySquenceOfBST(int [] sequence) {
        if(sequence.length == 0)return false;
         return helper(sequence,0,sequence.length-1);
    }
}
```

#### 复杂链表的复制

```java
/*
public class RandomListNode {
    int label;
    RandomListNode next = null;
    RandomListNode random = null;

    RandomListNode(int label) {
        this.label = label;
    }
}
*/
public class Solution {
    public RandomListNode Clone(RandomListNode pHead)
    {
      /*
      *1、遍历链表，复制每个结点，如复制结点A得到A1，将结点A1插到结点A后面；
      *2、重新遍历链表，复制老结点的随机指针给新结点，如A1.random = A.random.next;
      *3、拆分链表，将链表拆分为原链表和复制后的链表
      */
        if(pHead == null)return null;
        RandomListNode cur = pHead;
        while(cur != null){
            RandomListNode node = new RandomListNode(cur.label);
            RandomListNode next = cur.next;
            cur.next = node;
            node.next = next;
            cur = next;
        }
        cur = pHead;
        while(cur != null){
            cur.next.random = cur.random == null? null:cur.random.next;
            cur = cur.next.next;//相当于之前的cur = cur.next,因为一定有后面的克隆节点
        }
        cur = pHead;
        RandomListNode res = pHead.next;
        while(cur != null){
            RandomListNode cloneNode = cur.next;
            cur.next = cloneNode.next;
            cloneNode.next = cur.next == null? null:cur.next.next;
            
            cur = cur.next;
        }
        return res;
    }
}
```

#### 二叉搜索树与双向链表

```java
public class Solution {
    TreeNode head,tail;
    public TreeNode Convert(TreeNode pRootOfTree) {
        inOrder(pRootOfTree);
        return head;
    }
    public void inOrder(TreeNode root){
        if(root == null)return;
        inOrder(root.left);
        if(head == null){
            head = root;
            tail = root;
        }else{
            tail.right = root;
            root.left = tail;
            tail = tail.right;
        }
        inOrder(root.right);
    }
}
```

#### 把数组排成最小的数

```java
import java.util.ArrayList;
import java.util.Arrays;
public class Solution {
    public String PrintMinNumber(int [] numbers) {
        String[] str = new String[numbers.length];
        
        for(int i = 0; i < numbers.length; i++)
            str[i] = String.valueOf(numbers[i]);
        
        Arrays.sort(str,(a,b)->(a+b).compareTo(b+a));
        
        StringBuilder builder = new StringBuilder();
        for(String s : str)
            builder.append(s);
        
        return builder.toString();
    }
}
```

#### 两个链表的第一个公共结点

```java
public class Solution {
    public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
        ListNode p1 = pHead1,p2 = pHead2;
        while(p1 != p2){
            p1 = p1 == null? pHead2 : p1.next;
            p2 = p2 == null? pHead1 : p2.next;
        }
        return p1 == p2? p1:null;
    }
}
```

#### 数字在排序数组中出现的次数

```java
public class Solution {
    int count;
    public void binarySearch(int[] array,int left ,int right,int k){
        if(left >= right) return;
        int mid = (left + right)/2;
        if(array[mid] == k){
            count ++;
            binarySearch(array,left,mid,k);
            binarySearch(array,mid+1,right,k);
        }else if(array[mid] < k){
            binarySearch(array,mid+1,right,k);
        }else{
            binarySearch(array,left,mid,k);
        }
    }
    public int GetNumberOfK(int [] array , int k) {
       binarySearch(array,0,array.length,k);
        return count;
    }
}
```

```java
public class Solution {
    int count;
    public void binarySearch(int[] array,int left ,int right,int k){
        if(left > right) return;
        int mid = (left + right)/2;
        if(array[mid] == k){
            count ++;
            binarySearch(array,left,mid-1,k);
            binarySearch(array,mid+1,right,k);
        }else if(array[mid] < k){
            binarySearch(array,mid+1,right,k);
        }else{
            binarySearch(array,left,mid-1,k);
        }
    }
    public int GetNumberOfK(int [] array , int k) {
       binarySearch(array,0,array.length-1,k);
        return count;
    }
}
```

#### 链表中环的入口结点

```java
public class Solution {

    public ListNode EntryNodeOfLoop(ListNode pHead)
    {
        ListNode fast = pHead, slow = pHead;
        while (true) {
            if (fast == null || fast.next == null) return null;
            fast = fast.next.next;
            slow = slow.next;
            if (fast == slow) break;
        }
        fast = pHead;
        while (slow != fast) {
            slow = slow.next;
            fast = fast.next;
        }
        return fast;
    }
}
```

#### 字符串的排列

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Collections;
public class Solution {
    public void swap(char[] ch,int i,int j){
        char c = ch[i];
        ch[i] = ch[j];
        ch[j] = c;
    }
    public void permutationHelper(char[] ch,int start,List<String> res){
        if(start >= ch.length){
            String s = String.valueOf(ch);
            if(!res.contains(s))
                res.add(s);
            return;
        }
        
        for(int i = start; i < ch.length; i++){
            swap(ch,i,start);
            permutationHelper(ch,start+1,res);//注意这里不是i+1
            swap(ch,i,start);
        }
    }
    public ArrayList<String> Permutation(String str) {
        if(str == null || str.equals("") == true)return new ArrayList<>();
        List<String> res = new ArrayList<>();
        
        permutationHelper(str.toCharArray(),0,res);
        Collections.sort(res);
        return (ArrayList)res;
    }
}
```

#### 和为S的连续正数序列

```java
import java.util.ArrayList;
public class Solution {
    public ArrayList<ArrayList<Integer> > FindContinuousSequence(int sum) {
       //双指针
        ArrayList<ArrayList<Integer>>res = new ArrayList<>();
        int low = 1,high = 2;
        while(low < high){
            int tmp_sum = (low + high) * (high - low + 1)  / 2;
            
            if(tmp_sum == sum){
                ArrayList<Integer>list = new ArrayList<>();
                for(int i = low; i <= high; i++)
                    list.add(i);
                res.add(list);
                low ++;
            }else if(tmp_sum < sum)
                high ++;
            else
                low ++;
        }
        
        return res;
    }
}
```

#### 和为S的两个数字

```java
import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> FindNumbersWithSum(int [] array,int sum) {
       ArrayList<Integer> list = new ArrayList<>();
        int low = 0,high = array.length-1;
       if (array == null || array.length < 2)
            return list;
       while(low < high){
           int tmp_sum = array[low] + array[high];
           if(tmp_sum > sum)
               high--;
           else if(tmp_sum < sum)
               low ++;
           else{
               list.add(array[low]);
               list.add(array[high]);
               return list;
           }
       }
        return list;
    }
}
```

#### 反转单词顺序列

```java
public class Solution {
    public String ReverseSentence(String str) {
        if(str == null || str.length() == 0)return str;
        if(str.trim().equals(""))return str;
        StringBuilder builder = new StringBuilder();
        String[] s = str.split(" ");
        
        for(int i = s.length-1; i >= 0; i--){
            builder.append(s[i]);
            builder.append(" ");
        }
        builder.delete(str.length(),str.length()+1);
        
        return builder.toString();
    }
}
```

#### 整数中1出现的次数

```java
public class Solution {
    public int NumberOf1Between1AndN_Solution(int n) {
        int count = 0;
        for(long i = 1; i <= n; i *= 10){
            int a = n / (int)i,b = n % (int)i;
            if(a % 10 == 0)
                count += (a/10) * i;
            else if(a % 10 == 1)
                count += (a/10) * i + b+1;
            else
                count += (a/10 + 1)*i;
        }
        return count;
    }
}
```

首先11算出现两次1，从个位开始判断，当每位为1时总共有多少数字。百位为例，对31456来说，a=314，b=56，百位为4大于0，若百位为1时，千位万位可能为a/10+1，即从0-31共32个，而个位十位从0-99共100个，所以未(a/10+1)*i；对311456来说，百位为1时，千位万位为0-30，因为31的时候十位个位不足100个，所以单独加上b+1；

#### 丑数

```java
import java.util.ArrayList;
public class Solution {
    public int GetUglyNumber_Solution(int index) {
       if(index <= 0)return 0;
        ArrayList<Integer>list = new ArrayList<>();
        list.add(1);
        int cou2 = 0,cou3 = 0,cou5 = 0;
        while(-- index > 0){
            int num2 = list.get(cou2) * 2;
            int num3 = list.get(cou3) * 3;
            int num5 = list.get(cou5) * 5;
            int min = Math.min(num2,Math.min(num3,num5));
            
            list.add(min);
            if(min == num2) cou2++;
            if(min == num3) cou3++;//注意这里不能用ifelse，因为可能三个队列的最小值有相同的
            if(min == num5) cou5++;
        }
        
        return list.get(list.size()-1);
    }
}
```

#### 数组中只出现一次的数字

```java
//num1,num2分别为长度为1的数组。传出参数
//将num1[0],num2[0]设置为返回结果
public class Solution {
    public int findBitIndex(int bitResult){
        int index = 0;
        while((bitResult & 1) == 0&&index < 32){
            index ++;
            bitResult >>= 1;
        }
        return index;
    }
    public boolean isBitOne(int target,int index){
        return ((target >> index)&1) == 1;
    }
    public void FindNumsAppearOnce(int [] array,int num1[] , int num2[]) {
        if(array.length == 2){
            num1[0] = array[0];
            num2[0] = array[1];
            return ;
        }
        int bitResult = 0;
        for(int i = 0; i < array.length; i++)
            bitResult ^= array[i];
        
        int bitIndex = findBitIndex(bitResult);
        
        for(int i = 0; i < array.length; i++){
            if(isBitOne(array[i],bitIndex))
                num1[0] ^= array[i];
            else
                num2[0] ^= array[i];
        }
    }
}
```

位运算中异或的性质：两个相同数字异或=0，一个数和0异或还是它本身。

当只有一个数出现一次时，我们把数组中所有的数，依次异或运算，最后剩下的就是落单的数，因为成对儿出现的都抵消了。

依照这个思路，我们来看两个数（我们假设是AB）出现一次的数组。我们首先还是先异或，剩下的数字肯定是A、B异或的结果，这个结果的二进制中的1，表现的是A和B的不同的位。我们就取第一个1所在的位数，假设是第3位，接着把原数组分成两组，分组标准是第3位是否为1。如此，相同的数肯定在一个组，因为相同数字所有位都相同，而不同的数，肯定不在一组。然后把这两个组按照最开始的思路，依次异或，剩余的两个结果就是这两个只出现一次的数字。 

#### 扑克牌顺子

```java
public class Solution {
    public boolean isContinuous(int [] numbers) {
        //大小王为0，万能牌
        if(numbers.length == 0)return false;
        int max = -1,min = 14;
        int[] count = new int[14];
        for(int i = 0; i < numbers.length; i++){
            if(numbers[i] == 0)continue;
            if(max < numbers[i])max = numbers[i];
            if(min > numbers[i])min = numbers[i];
            if(++count[numbers[i]] > 1)return false;//除了0没有重复
        }
        if(max - min < 5)return true;//max - min < 5
        return false;
    }
}
```

#### 不用加减乘除做加法

```java
public class Solution {
    public int Add(int num1,int num2) {
        //异或操作=相加但是没有进位
        //相与向左移一位=进位值
        while(num2 != 0){//进位不为0
            int tmp = num1 ^ num2;//相加结果
            num2 = (num1 & num2) << 1;//进位结果
            num1 = tmp;//相加结果放在num1中
        }
        return num1;
    }
}
```

首先看十进制是如何做的： 5+7=12，三步走
第一步：相加各位的值，不算进位，得到2。
第二步：计算进位值，得到10. 如果这一步的进位值为0，那么第一步得到的值就是最终结果。

第三步：重复上述两步，只是相加的值变成上述两步的得到的结果2和10，得到12。

同样我们可以用三步走的方式计算二进制值相加： 5-101，7-111 第一步：相加各位的值，不算进位，得到010，二进制每位相加就相当于各位做异或操作，101^111。

第二步：计算进位值，得到1010，相当于各位做与操作得到101，再向左移一位得到1010，(101&111)<<1。

第三步重复上述两步， 各位相加 010^1010=1000，进位值为100=(010&1010)<<1。
     继续重复上述两步：1000^100 = 1100，进位值为0，跳出循环，1100为最终结果。

#### 删除链表中重复的结点

```java
public class Solution {
    public ListNode deleteDuplication(ListNode pHead)
    {
        ListNode dummy = new ListNode(0);
        ListNode pre = dummy,cur = pHead;
        while(cur != null){
            if(cur.next != null && cur.val == cur.next.val){
                while(cur.next != null && cur.val == cur.next.val)
                    cur = cur.next;
                cur = cur.next;
                continue;
            }
            pre.next = cur;
            pre = cur;
            cur = cur.next;
        }
        pre.next = cur;
        return dummy.next;
    }
}
```

#### 字符流中第一个不重复的字符

```java
public class Solution {
    //Insert one char from stringstream
    int[] count = new int[256];//0：没出现过，1：第一个第一次出现 2：第二个第一次出现 -1：出现了多次
    int index;//记录出现的次数
    public void Insert(char ch)
    {
        if(count[ch] == 0)
            count[ch] = ++index;
        else
            count[ch] = -1;
    }
  //return the first appearence once char in current stringstream
    public char FirstAppearingOnce()
    {
        int minCount = Integer.MAX_VALUE;
        char ch = '#';
        for(int i = 0; i < 256; i++){
            if(count[i] > 0 && minCount > count[i]){
                minCount = count[i];
                ch = (char)i;
            }
        }
        return ch;
    }
}
```

#### 二叉树的下一个结点

```java
/*
public class TreeLinkNode {
    int val;
    TreeLinkNode left = null;
    TreeLinkNode right = null;
    TreeLinkNode next = null;

    TreeLinkNode(int val) {
        this.val = val;
    }
}
*/
public class Solution {
    public TreeLinkNode GetNext(TreeLinkNode pNode)
    {
        if(pNode.right != null){ //如果有右子树，则找右子树的最左节点
            pNode = pNode.right;
            while(pNode.left != null)
                pNode = pNode.left;
            return pNode;
        }
        while(pNode.next != null){//没右子树，则找第一个当前节点是父节点左孩子的节点
            if(pNode.next.left == pNode)
                return pNode.next;
            pNode = pNode.next;
        }
        return null;
    }
}
```

#### 数组中的逆序对

```java
public class Solution {
    public int InverseHelper(int[] array,int start,int end){//start = 0,end = 7,mid = 3
        if(start >= end)
            return 0;
        int mid = start + (end - start)/2;
        
        int leftCount = InverseHelper(array,start,mid);
        int rightCount = InverseHelper(array,mid+1,end);
        
        int count = 0,copy_index = end-start;
        int[] copy = new int[end-start+1];
        
        int left = mid,right = end;
        while(left >= start && right >= mid+1){
            if(array[left] > array[right]){
                count = (count + (right-mid))%1000000007;
                copy[copy_index--] = array[left--];
            }
            else{
                copy[copy_index--] = array[right--];
            }
        }
        while(left >= start)
            copy[copy_index--] = array[left--];
        while(right >= mid+1)
            copy[copy_index--] = array[right--];
        for(int i = start; i <= end; i++)
            array[i] = copy[++copy_index];
        
        return (leftCount + rightCount + count)%1000000007;
    }
    public int InversePairs(int [] array) {
        return InverseHelper(array,0,array.length-1)%1000000007;
    }
}
```

#### 数组中重复的数字

```java
public class Solution {
    // Parameters:
    //    numbers:     an array of integers
    //    length:      the length of array numbers
    //    duplication: (Output) the duplicated number in the array number,length of duplication array is 1,so using duplication[0] = ? in implementation;
    //                  Here duplication like pointor in C/C++, duplication[0] equal *duplication in C/C++
    //    这里要特别注意~返回任意重复的一个，赋值duplication[0]
    // Return value:       true if the input is valid, and there are some duplications in the array number
    //                     otherwise false
    public boolean duplicate(int numbers[],int length,int [] duplication) {
        if(length <= 0)return false;
        
        for(int i = 0; i < length; i++){
            int index = numbers[i] % length;
            if(numbers[index] >= length){
                duplication[0] = index;
                return true;
            }
            numbers[index] += length;
        }
        return false;
    }
}
```

空间复杂度为O(1)

#### 剪绳子

```java
public class Solution {
    public int cutRope(int target) {
        if(target < 2)return 0;
        if(target == 2)return 1;
        if(target == 3)return 2;
        
        int count3 = target / 3;
        if(target - count3 * 3 == 1)count3 --;
        int count2 = (target - 3*count3)/2;
        
        return (int)Math.pow(3,count3) * (int)Math.pow(2,count2);
    }
}
```

```java
public class Solution {
    public int cutRope(int target) {
        //必须分
        if(target < 2)return 0;
        if(target == 2)return 1;
        if(target == 3)return 2;
        
        int[] dp = new int[target+1];
        dp[1] = 1;dp[2] = 2;dp[3] = 3;//已经分了以后的小段
        for(int i = 4; i <= target; i++){
            int max = 0;
            for(int j = 1; j <= i/2; j++){
                if(max < dp[i-j]*dp[j])
                    max = dp[j]*dp[i-j];
            }
            dp[i] = max;
        }
        
        return dp[target];
    }
}
```

注意4分和不分结果是一样的

#### 机器人的运动范围

```java
public class Solution {
    public int helper(int target){
        int sum = 0;
        while(target != 0){
            sum += target % 10;
            target /= 10;
        }
        return sum;
    }
    public int backtrack(int threshold,int rows,int cols,int max_rows,int max_cols,boolean[][] visited){
        
        if(rows < 0 || rows >= max_rows || cols < 0 || cols >= max_cols || 
           visited[rows][cols] == true || helper(rows)+helper(cols) > threshold)return 0;
        int count = 1;
        visited[rows][cols] = true;
        count += backtrack(threshold,rows-1,cols,max_rows,max_cols,visited);
        count += backtrack(threshold,rows+1,cols,max_rows,max_cols,visited);
        count += backtrack(threshold,rows,cols-1,max_rows,max_cols,visited);
        count += backtrack(threshold,rows,cols+1,max_rows,max_cols,visited);
        return count;
    }
    public int movingCount(int threshold, int rows, int cols)
    {
        boolean[][] visited = new boolean[rows][cols];
        return backtrack(threshold,0,0,rows,cols,visited);
    }
}
```

#### 矩阵中的路径

```java
public class Solution {
    public boolean backtrack(char[] matrix,int rows,int cols,int max_rows,int max_cols,char[] str,int index,boolean[] visited){
        if(index >= str.length)return true;
        if(rows < 0 || rows >= max_rows || cols < 0 || cols >= max_cols ||
          matrix[rows*max_cols + cols] != str[index] || visited[rows*max_cols + cols] == true)
            return false;
        
        visited[rows * max_cols + cols] = true;
        if(backtrack(matrix,rows+1,cols,max_rows,max_cols,str,index+1,visited)||
          backtrack(matrix,rows-1,cols,max_rows,max_cols,str,index+1,visited) ||
          backtrack(matrix,rows,cols+1,max_rows,max_cols,str,index+1,visited) ||
          backtrack(matrix,rows,cols-1,max_rows,max_cols,str,index+1,visited))
            return true;
        visited[rows * max_cols + cols] = false;
        return false;
    }
    public boolean hasPath(char[] matrix, int rows, int cols, char[] str)
    {
        boolean[] visited = new boolean[matrix.length];
        for(int i = 0; i < rows; i++){
            for(int j = 0; j < cols; j++){
                if(matrix[i*cols + j] == str[0] && backtrack(matrix,i,j,rows,cols,str,0,visited)){
                    return true;
                }
            }
        }
        return false;
    }
}
```

#### 滑动窗口的最大值

```java
import java.util.ArrayList;
import java.util.LinkedList;
public class Solution {
    public ArrayList<Integer> maxInWindows(int [] num, int size)
    {
        ArrayList<Integer> res_list = new ArrayList<>();
        LinkedList<Integer> max_list = new LinkedList<>();
        if(size == 0)return res_list;
        for(int i = 0; i < num.length; i++){
            while(!max_list.isEmpty() && num[max_list.getLast()] <= num[i])
                max_list.pollLast();
            max_list.add(i);
            if(max_list.peek() < i - size + 1){//每次左边少一个元素，用if判断max是否出去就可以，不用while
                max_list.pollFirst();
            }
            if(i >= size -1) res_list.add(num[max_list.peek()]);
        }
        return res_list;
    }
}
```

#### 序列化二叉树

```java
/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    int index = -1;
    String Serialize(TreeNode root) {
        if(root == null)
            return "#,";
        
        StringBuilder builder = new StringBuilder();
        builder.append(root.val + ",");
        builder.append(Serialize(root.left));
        builder.append(Serialize(root.right));
        
        return builder.toString();
  }
    TreeNode Deserialize(String str) {
        if(++index >= str.length())return null;//不能为index++，因为加完了后底下要用到，会越界
        
        String[] s = str.split(",");
        if(s[index].equals("#"))return null; 
        TreeNode node = new TreeNode(Integer.valueOf(s[index]));
        node.left = Deserialize(str);//遇到#不递归返回null
        node.right = Deserialize(str);
        return node;
  }
}
```

#### 构建乘积数组

```java
import java.util.ArrayList;
public class Solution {
    public int[] multiply(int[] A) {
        int[] B = new int[A.length];
        B[0] = 1;
        for(int i = 1; i < A.length; i++)//下三角
            B[i] = B[i-1] * A[i-1];
        int mul = 1;
        for(int i = A.length-2; i >= 0; i--){//下三角*上三角
            mul *= A[i+1];
            B[i] *= mul;
        }
        return B;
    }
}
```

#### 正则表达式匹配

```java
public class Solution {
    public boolean matchHelper(char[] str,int i,char[] pattern,int j){
        if(i == str.length && j == pattern.length)return true;
        if(i != str.length && j == pattern.length)return false;

        if(j+1 < pattern.length && pattern[j+1] == '*'){
            if((i < str.length && str[i] == pattern[j]) || i < str.length && pattern[j] == '.'){
                return matchHelper(str,i,pattern,j+2) ||//匹配零个字符
                       matchHelper(str,i+1,pattern,j+2) ||//匹配一个字符
                       matchHelper(str,i+1,pattern,j);//循环匹配一个字符
            }else{
                return matchHelper(str,i,pattern,j+2);//匹配零个字符
            }
        }else{
            if((i < str.length && str[i] == pattern[j]) || i < str.length && pattern[j] == '.')
                return matchHelper(str,i+1,pattern,j+1);//匹配一个字符
            else return false;
        }
            
    }
    public boolean match(char[] str, char[] pattern)
    {
        if(str == null || pattern == null)return false;
        return matchHelper(str,0,pattern,0);
    }
}
```

需要注意的是

1.没有i == str.length || j == pattern.length就返回false，因为会出现str = “”， pattern = “.*”的情况

2.因为没有判断i<str.length，在底下每一个要返回i+1或者i+2的if的里都要写i<str.length，因为i+1会越界

#### 表示数值的字符串

```java
public class Solution {
    int index = 0;
    public boolean scanUnsignedInteger(char[] str){
        int head = index;
        while(index < str.length && str[index] >= '0' && str[index] <= '9')
            index++;
        
        return index > head;
    }
    public boolean scanInteger(char[] str){
        if(index < str.length && (str[index] == '+' || str[index] == '-'))
            index ++;
        return scanUnsignedInteger(str);
    }
    public boolean isNumeric(char[] str) {
       if(str == null || str.length == 0) return false;
        boolean res = scanInteger(str);
        
        if(index < str.length && str[index] == '.'){
            index++;
            res = scanUnsignedInteger(str) || res;//可以没有前面的带符号整数部分直接有.加无符号整数
        }
        if(index < str.length && (str[index] == 'E' || str[index] == 'e')){
            index++;
            res = res && scanInteger(str);//有e的话后面必须要有带符号整数
        }
        
        return res && index == str.length;
    }
}
```

注意res = scanUnsignedInteger(str) || res 这里不能用 res ||  scanUnsignedInteger(str)，因为||的短路特性，123.45e+6，遇到.前res =  true,如果res放到前面，不会扫描45，而是直接res = true了，index也不动依旧指向数字4

#### 数据流中的中位数

```java
import java.util.PriorityQueue;
import java.util.Comparator;
public class Solution {
    int count;//heap中的数据个数
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(new Comparator<Integer>(){
        public int compare(Integer a,Integer b){//这里不能用int，因为必须为Object
            return b-a;
        }
    });
    public void Insert(Integer num) {
        if(count % 2 != 0){
            minHeap.offer(num);
            int min = minHeap.poll();
            maxHeap.offer(min);
        }else{
            /* 偶数个往maxHeap中放，运行时mixHeap的个数始终>=maxHeap
            count为偶数个 两个Heap一样大
            奇数个 minHeap比maxHeap多一个，多的那个就是作为头的中位数
            */
            maxHeap.offer(num);
            int max = maxHeap.poll();
            minHeap.offer(max);
        }
        count++;
    }

    public Double GetMedian() {
        if(count % 2 != 0)//奇数个取后半部分小顶堆的最小
            return (double)minHeap.peek();
        else//偶数个取前半部分最大和后半部分最小
            return (double)(maxHeap.peek() + minHeap.peek()) / 2;
    }
}
```

  例如，传入的数据为：[5,2,3,4,1,6,7,0,8],那么按照要求，输出是”5.00 3.50 3.00 3.50 3.00 3.50 4.00 3.50 4.00 “ 

  那么整个程序的执行流程应该是（用min表示小顶堆，max表示大顶堆）： 

- 5先进入大顶堆，将大顶堆中最大值放入小顶堆中，此时min=[5],max=[无]，avg=[5.00]  
- 2先进入小顶堆，将小顶堆中最小值放入大顶堆中，此时min=[5],max=[2],avg=[(5+2)/2]=[3.50]  
- 3先进入大顶堆，将大顶堆中最大值放入小顶堆中，此时min=[3,5],max=[2],avg=[3.00]  
- 4先进入小顶堆，将小顶堆中最小值放入大顶堆中，此时min=[4,5],max=[3,2],avg=[(4+3)/2]=[3.50]  
- 1先进入大顶堆，将大顶堆中最大值放入小顶堆中，此时min=[3,4,5],max=[2,1]，avg=[3/00]  
- 6先进入小顶堆，将小顶堆中最小值放入大顶堆中，此时min=[4,5,6],max=[3,2,1],avg=[(4+3)/2]=[3.50]  
- 7先进入大顶堆，将大顶堆中最大值放入小顶堆中，此时min=[4,5,6,7],max=[3,2,1],avg=[4]=[4.00]  
- 0先进入小顶堆，将小顶堆中最大值放入小顶堆中，此时min=[4,5,6,7],max=[3,2,1,0],avg=[(4+3)/2]=[3.50]  
- 8先进入大顶堆，将大顶堆中最小值放入大顶堆中，此时min=[4,5,6,7,8],max=[3,2,1,0],avg=[4.00]  