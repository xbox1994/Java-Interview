https://leetcode.com/problems/kth-largest-element-in-an-array/

Find the kth largest element in an unsorted array. Note that it is the kth largest element in the sorted order, not the kth distinct element.

Example 1:

```
Input: [3,2,1,5,6,4] and k = 2
Output: 5
```

Example 2:

```
Input: [3,2,3,1,2,4,5,5,6] and k = 4
Output: 4
```

Note:  
You may assume k is always valid, 1 ≤ k ≤ array's length.

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


欢迎光临[我的博客](http://www.wangtianyi.top/?utm_source=github&utm_medium=github)，发现更多技术资源~
