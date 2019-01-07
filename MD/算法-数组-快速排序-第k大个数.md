https://leetcode.com/problems/kth-largest-element-in-an-array/

Given two sorted integer arrays nums1 and nums2, merge nums2 into nums1 as one sorted array.

Note:

* The number of elements initialized in nums1 and nums2 are m and n respectively.
* You may assume that nums1 has enough space (size that is greater or equal to m + n) to hold additional elements from nums2.
Example:
```
Input:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3

Output: [1,2,2,3,5,6]
```

快排的每一次排序就是找到一个第N大的数，修改快排的递归逻辑。算法复杂度为O(N)。

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        return sort(nums, 0, nums.length-1, k);
    }
    
    private int sort(int[] nums, int left, int right, int k){
        int base = nums[left];
        int leftt = left;
        int rightt = right;
        while(leftt < rightt){
            while(leftt < rightt && nums[rightt] > base){
                rightt--;
            }
            if(leftt < rightt){
                int temp = nums[rightt];
                nums[rightt] = nums[leftt];
                nums[leftt] = temp;
                leftt++;
            }
            while(leftt < rightt && nums[leftt] < base){
                leftt++;
            }
            if(leftt < rightt){
                int temp = nums[rightt];
                nums[rightt] = nums[leftt];
                nums[leftt] = temp;
                rightt--;
            }
        }
         int rank = nums.length - leftt;
        if(rank == k){
            return nums[leftt];
        }else if(rank<k){
            return sort(nums, left, leftt-1, k);
        }else{
            return sort(nums, leftt+1, right, k);
        }
    }
}
```


欢迎光临[91Code](http://www.91code.info/?utm_source=github&utm_medium=github)，发现更多技术资源~
