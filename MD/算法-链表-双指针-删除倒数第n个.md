https://leetcode.com/problems/remove-nth-node-from-end-of-list/

给一个链表，删除倒数第n个节点

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        
        ListNode q = dummy, p = dummy;
        while(n-->-1){
            p = p.next;
        }
        while(p!=null){
            q = q.next;
            p = p.next;
        }
        q.next = q.next.next; // 优化：释放被删除节点
        return dummy.next;
    }
}
```

欢迎光临[我的博客](http://www.wangtianyi.top/?utm_source=github&utm_medium=github)，发现更多技术资源~
