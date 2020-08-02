---
title: LeetCode刷题记录-腾讯精选练习
date: 2019-10-19 12:00:00
tags:
  - LeetCode
---

#### 7.整数反转

```java
class Solution {
    public int reverse(int x) {
        int rx = 0,y = x;
        if(x < 0) x *= -1;
        while(x != 0){
            
            if(rx > (Integer.MAX_VALUE-x%10)/10)
                return 0;
            rx = rx * 10 + x % 10;
            x /= 10;
        }
        
        if(y < 0)
            return -1*rx;
        return rx;
    }
}
```

#### 8.字符串转换整数

```java
class Solution {
    public int myAtoi(String str) {
        int sum = 0, i = 0;
        boolean flag = false,minus = false;
        
        //找第一个非空格字符
        for(i = 0; i < str.length(); i++){
            char ch = str.charAt(i);
            if(ch == ' ')
                continue;
            else if(ch >= '0' && ch <= '9')break;
            else if(ch == '+' || ch == '-'){
                if(i == str.length()-1 || str.charAt(i+1) > '9' || str.charAt(i+1) < '0')return 0;
                else if(ch == '-')minus = true;
            }
            else return 0;
        }
        
        //开始转换
        for(int j = i; j < str.length(); j++){
            char ch = str.charAt(j);
            if(ch < '0' || ch > '9')break;
            if(sum > Integer.MAX_VALUE / 10)//注意这里的溢出判断
                return minus == false?Integer.MAX_VALUE : Integer.MIN_VALUE;
            else if(sum == Integer.MAX_VALUE /10){//注意这里的溢出判断
                if(minus == false && ch >= '7')
                    return Integer.MAX_VALUE;
                else if(minus == true && ch >= '8')
                    return Integer.MIN_VALUE;
            }
            sum = sum * 10 + ch - '0';
        }
        return minus == true? -1*sum:sum;
    }
}
```

#### 9.回文数

```java
class Solution {
    public boolean isPalindrome(int x) {
        if(x < 0 || (x % 10 == 0 && x != 0))
            return false;
        int rx = 0;
        while(x > rx){
            rx = rx * 10 + x % 10;
            x /= 10;
        }
        
        return x == rx || x == rx/10;
    }
}
```

#### 16.最接近的三数之和

```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        Arrays.sort(nums);
        int res = nums[0] + nums[1] + nums[2];
        for(int i = 0; i < nums.length; i++){
            int left = i+1;
            int right = nums.length-1;
            
            while(left < right){
                int sum = nums[i] + nums[left] + nums[right];
                if(Math.abs(target - sum) < Math.abs(target - res))
                    res = sum;
                
                if(target - sum > 0) left++;
                else if(target - sum < 0)right--;
                else return sum;
            }
        }
        return res;
    }
}
```

#### 43.字符串相乘

```java
class Solution {
    public String multiply(String num1, String num2) {
        String zero = "0";
        if(zero.equals(num1) || zero.equals(num2))
            return zero;
        int[] arr = new int [num1.length() + num2.length()];
        
        for(int i = num1.length()-1; i >= 0; i--){
            int num1_i = num1.charAt(i)-'0'; 
            for(int j = num2.length()-1; j >= 0; j--){
                int num2_j = num2.charAt(j)-'0';
                
                //arr[i+j+1] += num1_i * num2_j;
                //arr[i+j] += arr[i+j+1] / 10;
                //arr[i+j+1] %= 10;
                int sum = (arr[i+j+1] + num1_i * num2_j);
                arr[i+j+1] = sum % 10;
                arr[i+j] += sum / 10;
            }
        }
        
        StringBuilder builder = new StringBuilder();
        for(int i = 0; i < arr.length; i++){
            if(i == 0 && arr[i] == 0)continue;
            builder.append(arr[i]);
        }
        
        return builder.toString();
    }
}
```

#### 89.格雷编码

```java
class Solution {
    public List<Integer> grayCode(int n) {
        List<Integer> list = new ArrayList<>();
        list.add(0);
        for(int i = 0;i < n; i++){
            int add = (int)Math.pow(2,i);
            
            for(int j = list.size()-1; j >= 0; j--){
                list.add(list.get(j) + add);
            }
        }
        
        return list;
    }
}
```

#### 231.2的幂

```java
class Solution {
    public boolean isPowerOfTwo(int n) {
        if(n <= 0)return false;
        return (n & (n-1)) == 0;
    }
}
```

#### 235.二叉树的最近公共祖先

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
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        int parentVal = root.val;
        int pVal = p.val;
        int qVal = q.val;
        
        if(parentVal > p.val && parentVal > q.val){
            return lowestCommonAncestor(root.left,p,q);
        }else if(parentVal < p.val && parentVal < q.val){
            return lowestCommonAncestor(root.right,p,q);
        }else{
            return root;
        }
    }
}
```

#### 237.删除链表中的节点

```java
class Solution {
    public void deleteNode(ListNode node) {
        node.val = node.next.val;
        node.next = node.next.next;
    }
}
```

#### 292.Nim游戏

1-3个石头，我必赢

4个石头我必输，因为总是给对方留下1-3个石头

5-7个石头我必赢，因为我可以拿掉1-3个石头，让对手面对4个石头

8个石头我必输，因为总是给对方留下5-7个石头

9-11个石头我必赢，因为总是可以让对手面对8个石头

12个石头我必输，因为总是给对方留下9-11个石头

….同理4，12，16，20等4的倍数会输。