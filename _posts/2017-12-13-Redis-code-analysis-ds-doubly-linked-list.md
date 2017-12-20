---
    author: xiao.liang
    date: 2017-12-13
    title: "Redis源码分析(数据结构篇)——双向链表(doubly linked list)"
    categories: 
        - Tech
    tags:
        - Redis
---

该系列是Redis源码分析的第一部分，主要对Redis的一些数据结构进行解读。注意，这里的数据结构并不是Redis使用层面上的数据结构（string, list, set等），而是支撑这些“数据结构”实现的更加底层的数据结构。

这是该系列的第一篇文章，讲解的是Redis实现的双向链表。链表作为最基本的数据结构之一，想必大家都已经烂熟于心。那么Redis实现的链表有何高明之处呢？还是有的。简而言之，Redis的链表是一个双向的异质链表，可以存储任意数据类型。

## 结构定义

Redis的链表定义了如下三个数据结构：

```c
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedef struct listIter {
    listNode *next;
    int direction;
} listIter;

typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```

结合上面的代码，我们可以看到，一个链表包含了如下一些结构：
- 一个指向链表头的指针
- 一个指向链表尾的指针
- 链表的长度
- 三个函数指针，当链表存储的是自定义的类型时，要求用户提供该类型的复制、释放和匹配函数（dup, free, match）

而一个链表节点包含了：
- 一个指向前一节点的指针
- 一个指向后一节点的指针
- 指向实际存储数据的指针

除此之外，还定义了一个迭代器（iterator）用于遍历链表，它支持如下一些操作：
- listGetIterator, listReleaseIterator 生成/销毁一个迭代器
- listRewind, listRewindTail 重定位至头部/尾部
- listNext 返回下一个节点

## 操作实现

Redis的链表实现和算法书上的实现并无差异，加之链表确实太简单了=。= 所以就不在赘述啦。

整体来说，Redis的链表实现比较有意思的地方就是使用函数指针来支持任意类型的存储，用户只需要指定需要的函数即可，颇有一种C++继承的味道。想想自己本科写的异质链表，瞬间感觉low爆了QAQ