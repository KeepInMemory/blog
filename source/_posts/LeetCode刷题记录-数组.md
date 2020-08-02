---
title: LeetCode刷题记录-数组
date: 2019-09-05 12:00:00
tags:
  - LeetCode
---



#### 两数之和

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

------

```java
class Solution {
    public int maxArea(int[] height) {
        int head = 0,end = height.length - 1,maxArea = 0;
        
        while(head < end)
        {
            maxArea = Math.max(maxArea,Math.min(height[head],height[end])*(end-head));
            if(height[head] < height[end])
                head++;
            else
                end--;
        }
        return maxArea;
    }
}
```

使用双指针，通过一次遍历，时间复杂度为O(n)。

最大面积受制于纵坐标最小的边和横坐标里的距离，从横坐标最大的左右两边开始减小距离，当距离减小，纵坐标会进行变化。纵坐标该怎么变才能保证有最大的面积呢，需要将短的边向里移动，因为长的边受制于短的边，将短的边移动后虽然距离会变小但是短的边可能会变长 ，面积可能会增大，而将长的边往里移动的话面积一定减小。

------

#### 删除排序数组中的重复项

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int j=0,k=0;
        for(i = 0; i < nums.length ; ++ i){
            if(nums[j] == nums[k]) ++j;
            else nums[++k] = nums[j++];
        }
        return k+1;
    }
}
```

------

#### 最大子序和

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

------

#### 加一

```java
class Solution {
    public int[] plusOne(int[] digits) {
        for(int i = digits.length-1; i >=0 ; i --){
            digits[i] ++;
            digits[i] %=10;
            if(digits[i] != 0)//不为0说明不再进位
                return digits;
        }
        int[] result = new int[digits.length+1];
        result[0] = 1;
        return result;
        
    }
}
```

这个题一共三种情况，第一个是末位无进位，直接将最后一位加一，如12+1=13；第二种情况是末位进位，进到中间位置就停下了，或者进到头但数组长度不增加，如399+1=400；第三种是进位一直进到头，数组长度加一，如999+1=1000。

最后需要注意，新建的数组只需要将首元素赋值位1，其他位置默认初始化位0，不需要再单独赋值。

------

#### 合并两个有序数组

给定两个有序整数数组 nums1 和 nums2，将 nums2 合并到 nums1 中，使得 num1 成为一个有序数组。

初始化 nums1 和 nums2 的元素数量分别为 m 和 n。

你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。

输入:

nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6], n = 3

输出: [1,2,2,3,5,6]

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = m-1,j = n-1,k = m+n-1;
        while(i >=0 && j >= 0)
        {
            nums1[k--] = nums1[i] > nums2[j]?nums1[i--]:nums2[j--];
        }
        
        if(j >= 0)
         System.arraycopy(nums2,0,nums1,0,j+1);
    }
}
```

有序数组可以采用双指针方法达到O(m+n)的复杂度。

从后往前遍历，先放较大的元素，最后将nums2剩下的小的元素顺序填到num1的前面

System.arraycopy(nums2,0,nums1,0,j+1)从nums2数组下标0开始，复制给nums1的下标0位置开始，复制j+1个元素。

------

#### 求众数

```java
class Solution {
    public int majorityElement(int[] nums) {
        int count = 0;
        Integer candidate = null;
        for(int num:nums){
            if(count == 0)
                candidate = num;
            count += ((candidate == num)? 1:-1);
        }
        return candidate;
    }
}
```

Boyer-Moore 算法，如果把众数+1，其他数-1，全部加起来一定大于0。

首先选取第一个数作为候选者，往后遍历，如果和候选者相同，那么+1，否则就-1。如果一直是正数，那么就一直遍历。如果减到了0，那么前面遍历过的序列就可以忽略掉了，重新确定候选者，直至最后剩下的那个候选者。

------

#### 移动零

```java
class Solution {
    public void moveZeroes(int[] nums){
        int count = 0;
        for(int i = 0 ; i < nums.length; ++i){
            if(nums[i] == 0)
                count++;
            else
                nums[i-count] = nums[i];
        }
        for(int i = nums.length - count; i < nums.length; ++i){
            nums[i] = 0;
        }
    }
}
```

------

#### 找到所有数组中消失的数字

```java
class Solution {
    public static void swap(int[] nums,int i, int j){
        nums[i] = nums[i] ^ nums[j];
        nums[j] = nums[i] ^ nums[j];
        nums[i] = nums[i] ^ nums[j];
    }
    public List<Integer> findDisappearedNumbers(int[] nums) {
        for(int i = 0 ; i < nums.length; ++ i){
            while(nums[nums[i]-1] != nums[i]){
                swap(nums,i,nums[i]-1);
            }
        }
        List<Integer>list = new LinkedList<>();
        for(int i = 0 ; i < nums.length ; ++ i){
            if(nums[i] != i +1){
                list.add(i+1);
            }
        }
        return list;
    }
}
```

因为这里要求没有额外的空间，所以交换操作使用异或运算。把各个数进行交换，交换到它应该在的位置上，虽然会有重复的数字出现，但只要该在对应位置上的数字在就可以了，从头往后找，对应位置的数字不正确的就是缺少的数字。本来是一个萝卜一个坑，但有的萝卜不再，坑被别的萝卜占了，找的就是占错了的。

------

#### 旋转数组

```java
class Solution {
    public void swap(int []nums,int a,int b){
        nums[a] = nums[a] ^ nums[b];
        nums[b] = nums[a] ^ nums[b];
        nums[a] = nums[a] ^ nums[b];
    }
    public void revise(int[] nums,int a,int b){
        int start = a,end = b;
        while(start < end){
            swap(nums,start,end);
            start++;
            end--;
        }
    }
    public void rotate(int[] nums, int k) {
        k %= nums.length;
        revise(nums,0,nums.length-1);
        revise(nums,0,k-1);
        revise(nums,k,nums.length-1);
    }
}
```

反转法，在这个方法中，我们首先将所有元素反转。然后反转前 k 个元素，再反转后面 n−kn-k*n*−*k* 个元素，就能得到想要的结果。还需注意的是k可能会大于数组的长度，所以要%数组长。

------

#### 存在重复元素

```java
class Solution {
    public boolean containsNearbyDuplicate(int[] nums, int k) {
        HashSet<Integer> set = new HashSet<>();
        for(int i = 0; i < nums.length; i++) {
            if(set.contains(nums[i])) {
                return true;
            }
            set.add(nums[i]);
            if(set.size() > k) {
                set.remove(nums[i - k]);
            }
        }
        return false;
    }
}
```

维护一个哈希表，里面始终最多包含 k 个元素，当出现重复值时则说明在 k 距离内存在重复元素
每次遍历一个元素则将其加入哈希表中，如果哈希表的大小大于 k，则移除最前面的数字。

------

#### 三个数的最大乘积

```java
class Solution {
    public int maximumProduct(int[] nums) {
        Arrays.sort(nums);
        int max1 = nums[nums.length -1];
        int max2 = nums[nums.length -2];
        int max3 = nums[nums.length -3];
        int min1 = nums[0];
        int min2 = nums[1];
        int res1 = max1*max2*max3;
        int res2 = min1*min2*max1;
        return res1>res2?res1:res2;
    }
}
```

这里列举下所有的情况

全为正数则取最大的三个

有正数有负数选择最大的三个正数和正数加两个负数的最大值

最大的为0其余为负数选择0

全为负数选择最大的三个

综上，只有两种情况，要么最大的三个，要么一个最大带两个最小

------

#### 单调数列

```java
public boolean isMonotonic(int[] A) {
	int asc = 0, desc = 0;
	for (int i = 0; i < A.length - 1; i++) {
		if (A[i] <= A[i + 1]) {
			asc++;
		}
		if (A[i] >= A[i + 1]) {
			desc++;
		}
	}
	return asc == A.length - 1 || desc == A.length - 1;
}
```

------

#### 至少是其他数字两倍的最大数

```java
class Solution {
    public int dominantIndex(int[] nums) {
       int sec = 0,fir = 0;
        int firIndex = 0;
        
        for(int i = 0 ; i < nums.length; ++ i){
            if(fir < nums[i]){
                sec = fir;
                firIndex = i;
                fir = nums[i];
            }else if(sec < nums[i]){
                sec = nums[i];
            }
        }
        return fir >= 2*sec ? firIndex:-1;
    }
}
```

仔细想想，为什么我们要将最大值与所有元素进行比较呢？如果我们找到第二大的元素，将它的两倍的值与最大值进行比较，不就能证明最大值是否大于所有元素两倍的值吗