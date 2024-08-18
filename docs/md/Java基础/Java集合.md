作者： 陈汤姆
<br/>博客：[https://github.com/linglongchen/linglongchen.github.io](https://github.com/linglongchen/linglongchen.github.io)

>自我学习的积累！😄





- Java集合
    - List
        - ArrayList
            - 非线程安全
            - 基于**动态数组**实现
            - 适合随机访问，增删性能慢
        - LinkedList
            - 非线程安全
            - 基于**双向链表**实现
            - 适合插入和删除，随机访问性能较低
    - Map
        - HashMap
            - 非线程安全
            - 基于哈希表实现
            - 不保证元素的顺序，不允许重复元素
        - Hashtable
            - 线程安全
                - synchronized实现
            - 基于哈希表实现
            - 不保证元素的顺序，不允许重复元素
        - TreeMap
            - 基于红黑树实现
            - 非线程安全
            - 保证了按键的有序访问
        - ConcurrentHashMap
            - 线程安全
            - 基于哈希表实现
    - Set
        - TreeSet
            - 基于红黑树实现
            - 允许重复元素，提供了按排序顺序进行遍历的功能
        - HashSet
            - 基于哈希表实现
            - 不保证元素的顺序，不允许重复元素
    - Queue
        - PriorityQueue  
            - 队列中的元素根据优先级进行排序
            - 非线程安全
            - 不允许队列中存在重复元素
            - 二叉堆（默认情况下是最小堆）实现
    - Deque
        - ArrayDeque
            - 非线程安全
            - 双端队列
            - 基于动态循环数组实现
            - 遵循先进先出（FIFO）原则，也可以作为一个栈使用，遵循后进先出（LIFO）原则