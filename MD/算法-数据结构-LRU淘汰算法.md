https://leetcode.com/problems/lru-cache/

LRU，全称Least Recently Used，最近最少使用缓存。

```java
import java.util.HashMap;
import java.util.LinkedList;

class LRUCache {
    private HashMap<Integer, Integer> cacheMap = new HashMap<>();
    private LinkedList<Integer> recentlyList = new LinkedList<>();
    private int capacity;

    public LRUCache(int capacity) {
        this.capacity = capacity;
    }

    public int get(int key) {
        if (!cacheMap.containsKey(key)) {
            return -1;
        }
        recentlyList.remove((Integer) key);
        recentlyList.add(key);
        return cacheMap.get(key);
    }

    public void put(int key, int value) {
        if (cacheMap.containsKey(key)) {
            recentlyList.remove((Integer) key);
        }else if(cacheMap.size() == capacity){
            cacheMap.remove(recentlyList.removeFirst());
        }
        recentlyList.add(key);
        cacheMap.put(key, value);
    }

    public static void main(String[] args) {
        LRUCache2 cache = new LRUCache2(2);
        cache.put(1, 1);
        cache.put(2, 2);
        System.out.println(cache.get(1)); // returns 1
        cache.put(3, 3); // 驱逐 key 2
        System.out.println(cache.get(2)); // returns -1 (not found)
        cache.put(4, 4); // 驱逐 key 1
        System.out.println(cache.get(1)); // returns -1 (not found)
        System.out.println(cache.get(3)); // returns 3
        System.out.println(cache.get(4)); // returns 4
    }
}
```

欢迎光临[我的博客](http://www.wangtianyi.top/?utm_source=github&utm_medium=github)，发现更多技术资源~
