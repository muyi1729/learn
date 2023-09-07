# Redis

**Redis 是一个使用 C 语言开发的，开源的高性能非关系型（NoSQL）的键值对数据库**。与传统数据库不同的是 **Redis 的数据是存在内存中的** ，也就是它是内存数据库，所以读写速度非常快，因此 Redis 被广泛应用于缓存方向，每秒可以处理超过 10万次读写操作，是已知性能最快的Key-Value DB。

Redis 可以存储键和五种不同类型的值之间的映射。键的类型只能为字符串，值支持五种数据类型：字符串、列表、集合、散列表、有序集合。

**Redis 提供了多种数据类型来支持不同的业务场景。Redis 还支持事务 、持久化、Lua 脚本、多种集群方案。**

## Redis的应用场景

- **计数器**：可以对 String 进行自增自减运算，从而实现计数器功能。Redis 这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量。
- **缓存**：将热点数据放到内存中，设置内存的最大使用量以及淘汰策略来保证缓存的命中率。
- **会话缓存**：可以使用 Redis 来统一存储多台应用服务器的会话信息。当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及可伸缩性。
- **查找表**：例如 DNS 记录就很适合使用 Redis 进行存储。查找表和缓存类似，也是利用了 Redis 快速的查找特性。但是查找表的内容不能失效，而缓存的内容可以失效，因为缓存不作为可靠的数据来源。
- **其它**：Set 可以实现交集、并集等操作，从而实现共同好友等功能。ZSet 可以实现有序性操作，从而实现排行榜等功能。

## 为什么要用 Redis/为什么要用缓存？

速度快，完全基于内存，使用C语言实现，网络层使用epoll解决高并发问题，单线程模型避免了不必要的上下文切换及竞争条件；

与传统数据库不同的是 Redis 的数据是存在内存中的，所以读写速度非常快，因此 redis 被广泛应用于缓存方向，每秒可以处理超过 10万次读写操作，是已知性能最快的Key-Value DB。另外，Redis 也经常用来做分布式锁。除此之外，Redis 支持事务 、持久化、LUA脚本、LRU驱动事件、多种集群方案。

简单来说使用缓存主要是为了提升用户体验以及应对更多的用户。

下面主要从“高性能”和“高并发”这两点来看待这个问题。

**高性能：**

假如用户第一次访问数据库中的某些数据的话，这个过程是比较慢，毕竟是从硬盘中读取的。但是，如果说，用户访问的数据属于高频数据并且不会经常改变的话，那么我们就可以很放心地将该用户访问的数据存在缓存中。

**这样有什么好处呢？** 那就是保证用户下一次再访问这些数据的时候就可以直接从缓存中获取了。操作缓存就是直接操作内存，所以速度相当快。

不过，要保持数据库和缓存中的数据的一致性。 如果数据库中的对应数据改变的之后，同步改变缓存中相应的数据即可！

<img src="https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210413143031.png" style="zoom: 67%;" />

**高并发：**

一般像 MySQL 这类的数据库的 QPS 大概都在 1w 左右（4 核 8g） ，但是使用 Redis 缓存之后很容易达到 10w+，甚至最高能达到 30w+（就单机 redis 的情况，redis 集群的话会更高）。

> QPS（Query Per Second）：服务器每秒可以执行的查询次数；

所以，直接操作缓存能够承受的数据库请求数量是远远大于直接访问数据库的，所以我们可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库。进而，我们也就提高系统整体的并发。

<img src="https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210413143226.png" style="zoom:67%;" />

## redis和Memcached的区别和共同点

**共同点** ：

1. 都是基于内存的数据库，一般都用来当做缓存使用。
2. 都有过期策略。
3. 两者的性能都非常高。

**区别** ：

1. **Redis 支持更丰富的数据类型（支持更复杂的应用场景）**。Redis 不仅仅支持简单的 k/v 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储。Memcached 只支持最简单的 k/v 数据类型。
2. Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用,而 Memecache 把数据全部存在内存之中。
3. Redis 有灾难恢复机制。因为可以把缓存中的数据持久化到磁盘上。
4. Redis 在服务器内存使用完之后，可以将不用的数据放到磁盘上。但是，Memcached 在服务器内存使用完之后，就会直接报异常。
5. Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；但是 Redis 目前是原生支持 cluster 模式的。
6. Memcached 是多线程，非阻塞 IO 复用的网络模型；Redis 使用单线程的多路 IO 复用模型。（Redis 6.0 引入了多线程 IO ）
7. Redis 支持发布订阅模型、Lua 脚本、事务等功能，而 Memcached 不支持。并且，Redis 支持更多的编程语言。
8. Memcached 过期数据的删除策略只用了惰性删除，而 Redis 同时使用了惰性删除、定期删除与定时删除。

| redis                                 | Memcached                                    |
| ------------------------------------- | -------------------------------------------- |
| 内存高速数据库                        | 高性能分布式内存缓存数据库，可缓存图片、视频 |
| 支持hash、list、set、zset、string结构 | 只支持key-value结构                          |
| 将大部分数据放到内存                  | 全部数据放到内存中                           |
| 支持持久化、主从复制备份              | 不支持数据持久化及数据备份                   |
| 数据丢失可通过AOF恢复                 | 挂掉后，数据不可恢复                         |

**使用场景：**

- 如果有持久方面的需求或对数据类型和处理有要求的应该选择redis。 
- 如果简单的key/value 存储应该选择memcached。

# 数据类型

> [Redis五种数据结构详解（理论+实战）](https://mp.weixin.qq.com/s?__biz=MzU1MzE4OTU0OQ==&mid=2247484413&idx=1&sn=4510c1db5ea07c57be066d712fc3bbab&chksm=fbf7ea3fcc806329b3f673dc28e80b68808b2226b3fef5025233f157b40a99145733a0c9018d&mpshare=1&scene=24&srcid=&sharer_sharetime=1592238354557&sharer_shareid=b6cb64bedb11e9fa59d690ebb6387d93&key=b1d2507ae5a23ff6a5d994d537556b9215fd7adfc46e6f1e2bd9f4cf8bc061e7b5a61eeb7e65d578494df3fb4d6398bf8ae552ad22a545202fc7755e535c86642b513e462faec910c9ff99d6d431b9b014170c9031cf8487beb1b7bf45a33ef508a5c900a702a714002d50d5d57b8e541bec197e7e5d9549e854ca15195e423d&ascene=14&uin=MTA2NzM1MDAzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=AfCYrEUf2EMDXqbojGIovkM%3D&pass_ticket=TH75Is630WzYlopDgihuWVrpncyi4%2Ftbhsug%2BQ4TehbcEeoaZG7BjLjiFyAvS%2Brx&wx_header=0)

Redis支持的常用5种数据类型指的是value类型，分别为：**字符串String、列表List、哈希Hash、集合Set、有序集合Zset**。

## Redis 常见数据结构以及使用场景分析

Redis 主要有5种数据类型，包括String，List，Set，Zset，Hash。

由于Redis是基于标准C写的，只有最基础的数据类型，因此Redis为了满足对外使用的5种数据类型，开发了属于自己**独有的一套基础数据结构**，使用这些数据结构来实现5种数据类型。

Redis底层的数据结构包括：**简单动态数组SDS、链表、字典、跳跃链表、整数集合、压缩列表、对象。**

Redis为了平衡空间和时间效率，针对value的具体类型在底层会采用不同的数据结构来实现，其中哈希表和压缩列表是复用比较多的数据结构，如下图展示了对外数据类型和底层数据结构之间的映射关系：

![](https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210415094348.png)

从图中可以看到**ziplist压缩列表**可以作为Zset、Set、List三种数据类型的底层实现，看来很强大，压缩列表是一种为了节约内存而开发的且经过特殊编码之后的连续内存块顺序型数据结构，底层结构还是比较复杂的。

### Redis的SDS和C中字符串相比有什么优势？

在C语言中使用N+1长度的字符数组来表示字符串，尾部使用'\0'作为结尾标志，对于此种实现无法满足Redis对于安全性、效率、丰富的功能的要求，因此Redis单独封装了SDS简单动态字符串结构。

SDS的实现细节，找了github**最新的src/sds.h**的定义：

```c
typedef char *sds;

/*这个用不到 忽略即可*/
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};

/*不同长度的header 8 16 32 64共4种 都给出了四个成员
len：当前使用的空间大小；alloc去掉header和结尾空字符的最大空间大小
flags:8位的标记 下面关于SDS_TYPE_x的宏定义只有5种 3bit足够了 5bit没有用
buf:这个跟C语言中的字符数组是一样的，从typedef char* sds可以知道就是这样的。
buf的最大长度是2^n 其中n为sdshdr的类型，如当选择sdshdr16，buf_max=2^16。
*/
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};

#define SDS_TYPE_5  0
#define SDS_TYPE_8  1
#define SDS_TYPE_16 2
#define SDS_TYPE_32 3
#define SDS_TYPE_64 4
#define SDS_TYPE_MASK 7
#define SDS_TYPE_BITS 3
```

![](https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210415094916.png)

从图中可以知道sds本质分为三部分：**header、buf、null结尾符**，其中header可以认为是整个sds的指引部分，给定了使用的空间大小、最大分配大小等信息，看下sdshdr8的实例：

![](https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210415094939.png)

sds的优势：

- **O(1)获取长度**: C字符串需要遍历而sds中有len可以直接获得；
- **防止缓冲区溢出bufferoverflow**: 当sds需要对字符串进行修改时，首先借助于len和alloc检查空间是否满足修改所需的要求，如果空间不够的话，SDS会自动扩展空间，避免了像C字符串操作中的覆盖情况；
- **有效降低内存分配次数**：C字符串在涉及增加或者清除操作时会改变底层数组的大小造成重新分配、sds使用了空间预分配和惰性空间释放机制，说白了就是每次在扩展时是成倍的多分配的，在缩容是也是先留着并不正式归还给OS，这两个机制也是比较好理解的；
- **二进制安全**：C语言字符串只能保存ascii码，对于图片、音频等信息无法保存，sds是二进制安全的，写入什么读取就是什么，不做任何过滤和限制；

![](https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210415095035.png)

**应用场景** ：简单的键值对缓存，一般常用在需要计数的场景，比如用户的访问次数、热点文章的点赞转发数量等等。

### Redis的字典是如何实现的？简述渐进式rehash的过程

字典可以基于ziplist和hashtable来实现，只讨论基于hashtable实现的原理。

字典是个层次非常明显的数据类型，如图：

![](https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210415095326.png)

最新的src/dict.h源码定义：

```c
//哈希节点结构
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;

//封装的是字典的操作函数指针
typedef struct dictType {
    uint64_t (*hashFunction)(const void *key);
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);
} dictType;

/* This is our hash table structure. Every dictionary has two of this as we
 * implement incremental rehashing, for the old to the new table. */
//哈希表结构 该部分是理解字典的关键
typedef struct dictht {
    dictEntry **table;
    unsigned long size;
    unsigned long sizemask;
    unsigned long used;
} dictht;

//字典结构
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

C语言的好处在于定义必须是由最底层向外的，因此可以看到一个明显的层次变化：

![](https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210415095618.webp)

- **关于dictEntry**

  dictEntry是哈希表节点，也就是存储数据地方，其保护的成员有：key,v,next指针。key保存着键值对中的键，v保存着键值对中的值，值可以是一个指针或者是uint64_t或者是int64_t。next是指向另一个哈希表节点的指针，这个指针可以将多个哈希值相同的键值对连接在一次，以此来解决哈希冲突的问题。

  如图为两个冲突的哈希节点的连接关系：

  ![](https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210415095719.png)

- **关于dictht**

  从源码看哈希表包括的成员有table、size、used、sizemask。table是一个数组，数组中的每个元素都是一个指向dictEntry结构的指针， 每个dictEntry结构保存着一个键值对；size 属性记录了哈希表table的大小，而used属性则记录了哈希表目前已有节点的数量。sizemask等于size-1和哈希值计算一个键在table数组的索引，也就是计算index时用到的。

  ![](https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210415095849.png)

  如上图展示了一个大小为4的table中的哈希节点情况，其中k1和k0在index=2发生了哈希冲突，进行开链表存在，本质上是先存储的k0，k1放置是发生冲突为了保证效率直接放在冲突链表的最前面，因为该链表没有尾指针。

- **关于dict**

  从源码中看到dict结构体就是字典的定义，包含的成员有type，privdata、ht、rehashidx。其中dictType指针类型的type指向了操作字典的api，理解为函数指针即可，ht是包含2个dictht的数组，也就是字典包含了2个哈希表，rehashidx进行rehash时使用的变量，privdata配合dictType指向的函数作为参数使用，这样就对字典的几个成员有了初步的认识。

  ![](https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210415100015.png)

- **字典的哈希算法**

  ```c
  //伪码：使用哈希函数，计算键key的哈希值
  hash = dict->type->hashFunction(key);
  //伪码：使用哈希表的sizemask和哈希值，计算出在ht[0]或许ht[1]的索引值
  index = hash & dict->ht[x].sizemask;
  //源码定义
  #define dictHashKey(d, key) (d)->type->hashFunction(key)
  ```

  redis使用MurmurHash算法计算哈希值，该算法最初由Austin Appleby在2008年发明，MurmurHash算法的无论数据输入情况如何都可以给出随机分布性较好的哈希值并且计算速度非常快，目前有MurmurHash2和MurmurHash3等版本。

- **普通Rehash重新散列**

  哈希表保存的键值对数量是**动态变化**的，为了让哈希表的负载因子维持在一个合理的范围之内，就需要对哈希表进行扩缩容。

  扩缩容是通过执行rehash重新散列来完成，对字典的哈希表执行普通rehash的基本步骤为**分配空间->逐个迁移->交换哈希表**，详细过程如下：

  1. 为字典的ht[1]哈希表分配空间，分配的空间大小取决于要执行的操作以及ht[0]当前包含的键值对数量：
     扩展操作时ht[1]的大小为第一个大于等于ht[0].used*2的2^n；
     收缩操作时ht[1]的大小为第一个大于等于ht[0].used的2^n ；

     **扩展时比如h[0].used=200，那么需要选择大于400的第一个2的幂，也就是2^9=512。**

  2. 将保存在ht[0]中的所有键值对重新计算键的哈希值和索引值rehash到ht[1]上；

  3. 重复rehash直到ht[0]包含的所有键值对全部迁移到了ht[1]之后释放 ht[0]， 将ht[1]设置为 ht[0]，并在ht[1]新创建一个空白哈希表， 为下一次rehash做准备。

- **渐进Rehash过程**

  Redis的rehash动作并不是一次性完成的，而是分多次、渐进式地完成的，原因在于当哈希表里保存的键值对数量很大时， 一次性将这些键值对全部rehash到ht[1]可能会导致服务器在一段时间内停止服务，这个是无法接受的。

  针对这种情况Redis采用了渐进式rehash，过程的详细步骤：

  1. 为ht[1]分配空间，这个过程和普通Rehash没有区别；
  2. 将rehashidx设置为0，表示rehash工作正式开始，同时这个rehashidx是递增的，从0开始表示从数组第一个元素开始rehash。
  3. 在rehash进行期间，每次对**字典执行增删改查操作**时，**顺带**将ht[0]哈希表在rehashidx索引上的键值对rehash到 ht[1]，完成后将rehashidx加1，指向下一个需要rehash的键值对。
  4. 随着字典操作的不断执行，最终ht[0]的所有键值对都会被rehash至ht[1]，再将rehashidx属性的值设为-1来表示 rehash操作已完成。

  渐进式 rehash的思想在于**将rehash键值对所需的计算工作分散到对字典的每个添加、删除、查找和更新操作上，从而避免了集中式rehash而带来的阻塞问题**。

### list

**list** 即是 **链表**。链表是一种非常常见的数据结构，特点是易于数据元素的插入和删除并且可以灵活调整链表长度，但是链表的随机访问困难。许多高级编程语言都内置了链表的实现，但是 C 语言并没有实现链表，所以 Redis 实现了自己的链表数据结构。Redis 的 list 的实现为一个 **双向链表**，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。

**应用场景:** 存储列表型数据结构，例如：评论列表、商品列表；发布与订阅或者说消息队列、慢查询。

### hash

hash 类似于 HashMap，内部实现也差不多(数组 + 链表)。不过，Redis 的 hash 做了更多优化。另外，hash 是一个 string 类型的 field 和 value 的映射表，**特别适合用于存储对象**，后续操作的时候，你可以直接仅仅修改这个对象中的某个字段的值。 比如我们可以 hash 数据结构来存储用户信息，商品信息等等。

**应用场景:** 系统中对象数据的存储。

### set

set 类似于 Java 中的 `HashSet` 。Redis 中的 set 类型是一种无序集合，集合中的元素没有先后顺序。当你需要存储一个列表数据，又不希望出现重复数据时，set 是一个很好的选择，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。可以基于 set 轻易实现交集、并集、差集的操作。比如：你可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis 可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能。这个过程也就是求交集的过程。

**应用场景:** 需要存放的数据不能重复以及需要获取多个数据源交集和并集等场景

### zset

和 set 相比，zset 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列，还可以通过 score 的范围来获取元素的列表。

**应用场景：** 需要对数据根据某个权重进行排序的场景。比如在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息。

## Zset底层实现？跳表搜索插入删除过程？

> 有序集合对象数据量少的时候使用压缩链表 ziplist 实现，有序集合使用两个紧挨在一起的压缩列表节点来保存，第一个节点保存元素的成员 member，第二个保存元素的分值 score。ziplist 内的集合元素按 score 从小到大排序，score 较小的排在表头位置。
>
> [ziplist 压缩列表](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMTI2Ng==&mid=2247484134&idx=1&sn=aa2ac7c6fc3c702ec3ea8fc22cff88b9&chksm=e9d0ccabdea745bdd45f56cece2c3ad12900d4104073e4590bec2cfa48ad6a765e5abb7c539f&scene=0&xtrack=1&key=349f5f7a80de3dfcc51dd31272c39c5bf5f83eceb7459c56a422b6ee02ccf9580629bc87e866d2803646afa95b25bcfdb92407f545ca1f4d2ed673de4f8d907d2793d555f6fe1cb65806edd63bbcba1dc76ade41fd8457f26b4efa8f097aeb517a38ff87dafd09c036526ddaf73f3e06c3dfacc1d34ecaf299ad107508fedf22&ascene=14&uin=MTA2NzM1MDAzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=AUP7MGb%2FOGtsGWxXvcj2JwA%3D&pass_ticket=36BRpqMNJifb34ESQBgqBJSQWrtLUahK1TBlGqaasUB3Jiudbkafjx6qOwvSMlMX&wx_header=0)

有序集合数据量大的时候使用 zset 结构作为底层实现，一个 zset 结构同时包含一个字典和一个跳跃表。查找删除插入成员的时间复杂度都是O(logN)，查找给定成员的分值为 O(1)。

跳表(skip List)是一种随机化的数据结构，基于并联的链表，实现简单，插入、删除、查找的复杂度均为O(logN)。简单说来跳表也是链表的一种，只不过它在链表的基础上增加了跳跃功能，正是这个跳跃的功能，使得在查找元素时，跳表能够提供O(logN)的时间复杂度。

**搜索**

跳表按 score 从小到大保存所有集合元素，查找时间复杂度为平均 O(logN)，最坏 O(N) 。

**插入**

之前就说了，之所以选用链表作为底层结构支持，也是为了高效地动态增删。单链表在知道删除的节点是谁时，时间复杂度为O(1)，因为跳表底层的单链表是有序的，为了维护这种有序性，在插入前需要遍历链表，找到该插入的位置，单链表遍历查找的时间复杂度是O(n)，同理可得，跳表的遍历也是需要遍历索引数，所以是O(logn)。

**删除**

删除的节点要分两种情况，如果该节点还在索引中，那删除时不仅要删除单链表中的节点，还要删除索引中的节点；另一种情况是删除的节点只在链表中，不在索引中，那只需要删除链表中的节点即可。但针对单链表来说，删除时都需要拿到前驱节点才可改变引用关系从而删除目标节点。

> [深入理解跳跃链表[一]](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMTI2Ng==&mid=2247483919&idx=1&sn=1b62c9a125be1bed7970aae906639d21&scene=21#wechat_redirect)
>
> [深入理解跳表在Redis中的应用](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMTI2Ng==&mid=2247483948&idx=1&sn=83c1ef41690a60d0b70fa4374d6e64b8&scene=21#wechat_redirect)

# 持久化

> [理解Redis持久化](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMTI2Ng==&mid=2247483700&idx=1&sn=a935b0320f9daaa3d0102c8a6095fad7&scene=21#wechat_redirect)

## 什么是Redis持久化？

持久化就是把内存的数据写到磁盘中去，防止服务宕机了内存数据丢失。

## Redis 的持久化机制是什么？各自的优缺点？

Redis 不同于 Memcached 的很重要一点就是，Redis 支持持久化，而且支持两种不同的持久化操作。**Redis 的一种持久化方式叫快照（snapshotting，RDB），另一种方式是只追加文件（append-only file, AOF）**。

### RDB持久化

RDB 是Redis DataBase缩写快照，是 Redis 默认的持久化方式。按照一定的时间将内存的数据以快照的形式保存到硬盘中，对应产生的数据文件为 dump.rdb。通过配置文件中的save参数来定义快照的周期。

![](https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210414144927.png)

优点：

- 只有一个文件 dump.rdb，方便持久化
- 容灾性好，一个文件可以保存到安全的磁盘
- 性能最大化，fork 子进程来完成写操作，让主进程继续处理命令，所以是 IO 最大化。使用单独子进程来进行持久化，主进程不会进行任何 IO 操作，保证了 redis 的高性能
- 相对于数据集大时，比 AOF 的启动效率更高

缺点：

- 数据安全性低。RDB 是间隔一段时间进行持久化，如果持久化之间 redis 发生故障，会发生数据丢失。所以这种方式更适合数据要求不严谨的时候

### AOF持久化

AOF持久化（即Append Only File持久化)，则是将Redis执行的每次写命令记录到单独的日志文件中，当重启Redis会重新将持久化的日志中文件恢复数据。

当两种方式同时开启时，数据恢复Redis会优先选择AOF恢复。

优点：

- 数据安全，aof 持久化可以配置 appendfsync 属性，有 always，每进行一次命令操作就记录到 aof 文件中一次。
- 通过 append 模式写文件，即使中途服务器宕机，可以通过 redis-check-aof 工具解决数据一致性问题。
- AOF 机制的 rewrite，通过该功能，Redis可以创建一个新的AOF文件来替代现有的AOF文件，新旧两个文件所保存的数据库状态相同，但新AOF文件不会包含任何浪费空间的冗余命令

缺点：

- AOF 文件比 RDB 文件大，且恢复速度慢。
- 数据大的时候，比 rdb 启动效率低。

**AOF 重写**：

AOF 重写可以产生一个新的 AOF 文件，这个新的 AOF 文件和原有的 AOF 文件所保存的数据库状态一样，但体积更小。

AOF 重写是一个有歧义的名字，该功能是通过读取数据库中的键值对来实现的，程序无须对现有 AOF 文件进行任何读入、分析或者写入操作。

在执行 BGREWRITEAOF 命令时，Redis 服务器会维护一个 AOF 重写缓冲区，该缓冲区会在子进程创建新 AOF 文件期间，记录服务器执行的所有写命令。当子进程完成创建新 AOF 文件的工作之后，服务器会将重写缓冲区中的所有内容追加到新 AOF 文件的末尾，使得新旧两个 AOF 文件所保存的数据库状态一致。最后，服务器用新的 AOF 文件替换旧的 AOF 文件，以此来完成 AOF 文件重写操作。

# Redis单线程模型

> [理解Redis单线程运行模式](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMTI2Ng==&mid=2247483755&idx=1&sn=20496502810f53409b1274afcc76a997&scene=21#wechat_redirect)
>
> [理解Redis的反应堆模式](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMTI2Ng==&mid=2247483747&idx=1&sn=184f86ec27472736f9d0f808b54fe753&scene=21#wechat_redirect)

## Redis线程模型

Redis基于Reactor模式开发了网络事件处理器，这个处理器被称为文件事件处理器（file event handler）。它的组成结构为4部分：多个套接字、IO多路复用程序、文件事件分派器、事件处理器。因为文件事件分派器队列是单线程的，所以Redis才叫单线程模型。

- 文件事件处理器使用 I/O 多路复用（multiplexing）程序来同时监听多个套接字， 并根据套接字目前执行的任务来为套接字关联不同的事件处理器。
- 当被监听的套接字准备好执行连接应答（accept）、读取（read）、写入（write）、关闭（close）等操作时， 与操作相对应的文件事件就会产生， 这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件。

虽然文件事件处理器以单线程方式运行， 但通过使用 I/O 多路复用程序来监听多个套接字， 文件事件处理器既实现了高性能的网络通信模型， 又可以很好地与 redis 服务器中其他同样以单线程方式运行的模块进行对接， 这保持了 Redis 内部单线程设计的简单性。

## redis是单线程还是多线程？为什么那么快？

> [硬核！15张图解Redis为什么这么快](https://mp.weixin.qq.com/s?__biz=MzU1MzE4OTU0OQ==&mid=2247488282&idx=1&sn=946fb9f88abaeeb3be24645f4c8e1684&chksm=fbf7fad8cc8073ce463384ddc529c109b0043349a061b8398e3c104ed9dad8bc2e4afb567fd0&mpshare=1&scene=24&srcid=0311atlg0RjJ8lHERi6N6RRZ&sharer_sharetime=1615477127756&sharer_shareid=b6cb64bedb11e9fa59d690ebb6387d93&key=79b605cf33a459ebee96b43016e49307552674269595790cb447180e788b12907ba91b96ddf328ddddfc61517ac400833532922ae49241c48b409e0f0b01acc1a4f1809bff33f02b01847563063eb19d4d6dab1a7752071e11d2d0f44995f4d3325fb87ead150e5b8dc99164ca7c801981e19a12f6c05b31e14adea895cd8a87&ascene=14&uin=MTA2NzM1MDAzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=AclwIjOyAmPPTPDX%2BQUzsI4%3D&pass_ticket=TH75Is630WzYlopDgihuWVrpncyi4%2Ftbhsug%2BQ4TehbcEeoaZG7BjLjiFyAvS%2Brx&wx_header=0)

redis是单线程，快的原因：

- 完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。数据存在内存中，类似于 HashMap，查找和操作的时间复杂度都是 O(1)
- 数据结构简单，对数据操作也简单，Redis 中的数据结构是专门进行设计的
- 采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗
- 使用多路 I/O 复用模型，非阻塞 IO


## Redis 没有使用多线程？为什么不使用多线程？

虽然说 Redis 是单线程模型，但是， 实际上，**Redis 在 4.0 之后的版本中就已经加入了对多线程的支持。**不过，Redis 4.0 增加的多线程主要是针对一些大键值对的删除操作的命令，使用这些命令就会使用主线程之外的其他线程来“异步处理”。

**为什么不使用多线程？**

1. 单线程编程容易并且更容易维护；
2. Redis 的性能瓶颈不在 CPU ，主要在内存和网络；
3. 多线程就会存在死锁、线程上下文切换等问题，甚至会影响性能。

## Redis6.0 之后为何引入了多线程？

**Redis6.0 引入多线程主要是为了提高网络 IO 读写性能**，因为这个算是 Redis 中的一个性能瓶颈（Redis 的瓶颈主要受限于内存和网络）。

虽然，Redis6.0 引入了多线程，但是 Redis 的多线程只是在网络数据的读写这类耗时操作上使用了， 执行命令仍然是单线程顺序执行。因此，你也不需要担心线程安全问题。

Redis6.0 的多线程默认是禁用的，只使用主线程。如需开启需要修改 redis 配置文件 `redis.conf` 。

# 数据过期

> [Redis的内存回收详解](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMTI2Ng==&mid=2247484148&idx=1&sn=ba1a92eb313b5bedf6ce05777a3a3f2c&chksm=e9d0ccb9dea745afadcfff2aac4252a94ead23678f3801d437952a152574a9a363aa8a4d8b13&scene=0&xtrack=1&key=47d47309d7f663133efb862a2704a2a82b7b97fcbab7fa0a6814d431b60c60cf989a5d1b987542963e5d63f0ff3ccee5629fb388ee7be22b907fb230784d677c7b45e56264a1d3c78e79707da713f97406fa44aa78f3b0db16f70922bda45b08c1cb292127e69a1fab9080c5ae4a06efa6cc2143b0c11a9bf61d240d9c5478c5&ascene=14&uin=MTA2NzM1MDAzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=AeIxHSGTrfPSokbhKRy2VBk%3D&pass_ticket=36BRpqMNJifb34ESQBgqBJSQWrtLUahK1TBlGqaasUB3Jiudbkafjx6qOwvSMlMX&wx_header=0)


## Redis 给缓存数据设置过期时间有啥用？

一般情况下，我们设置保存缓存数据的时候都会设置一个过期时间。为什么呢？

因为内存是有限的，如果缓存中的所有数据都是一直保存的话，分分钟直接Out of memory。

Redis 自带了给缓存数据设置过期时间的功能。


注意：**Redis中除了字符串类型有自己独有设置过期时间的命令 `setex` 外，其他方法都需要依靠 `expire` 命令来设置过期时间 。另外， `persist` 命令可以移除一个键的过期时间：**

**过期时间除了有助于缓解内存的消耗，还有什么其他用么？**

很多时候，我们的业务场景就是需要某个数据只在某一时间段内存在，比如我们的短信验证码可能只在1分钟内有效，用户登录的 token 可能只在 1 天内有效。如果使用传统的数据库来处理的话，一般都是自己判断过期，这样更麻烦并且性能要差很多。

## Redis是如何判断数据是否过期的呢？

Redis 通过一个叫做过期字典（可以看作是hash表）来保存数据过期的时间。过期字典的键指向Redis数据库中的某个key(键)，过期字典的值是一个long long类型的整数，这个整数保存了key所指向的数据库键的过期时间（毫秒精度的UNIX时间戳）。 

<img src="https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210414152513.png" style="zoom: 67%;" />

## 过期的数据的删除策略

Redis 是 key-value数据库，可以设置 Redis 中缓存的key的过期时间。Redis的过期策略就是指当Redis中缓存的key过期了，Redis如何处理。

常用的过期数据的删除策略就三个：

1. **定时删除**：在设置键的过期时间的同时，创建一个定时器，让定时器在键的过期时间来临时，立即执行对键的删除操作。该策略可以立即清除过期的数据，对内存很友好，但是会占用大量的CPU资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。
2. **惰性删除** ：只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。
3. **定期删除** ： 每隔一定的时间，会扫描一定数量的数据库的 expires 字典中一定数量的key，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源达到最优的平衡效果。

定期删除对内存更加友好，惰性删除对CPU更加友好。两者各有千秋，所以Redis 采用的是 **定期删除+惰性/懒汉式删除** 。

但是，仅仅通过给 key 设置过期时间还是有问题的。因为还是可能存在定期删除和惰性删除漏掉了很多过期 key 的情况。这样就导致大量过期 key 堆积在内存里，然后就Out of memory了。

怎么解决这个问题呢？答案就是： **Redis 内存淘汰机制。**

## Redis 内存淘汰机制

> 相关问题：MySQL 里有 2000w 数据，Redis 中只存 20w 的数据，如何保证 Redis 中的数据都是热点数据?

redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略。

Redis 提供 6 种数据淘汰策略：

设置过期时间的键空间选择性移除：

- **volatile-lru（**least recently used**）**：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
- **volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
- **volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰

全局的键空间选择性移除：

- **allkeys-lru（least recently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）
- **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰
- **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！

4.0 版本后增加以下两种：

- **volatile-lfu**：从已设置过期时间的数据集(server.db[i].expires)中挑选最不经常使用的数据淘汰
- **allkeys-lfu**：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 key

# Redis 事务

Redis事务功能是通过 MULTI、EXEC、DISCARD 和 WATCH 四个原语实现的。

Redis会将一个事务中的所有命令序列化，然后按顺序执行。

- **redis 不支持回滚**，“Redis 在事务失败时不进行回滚，而是继续执行余下的命令”， 所以 Redis 的内部可以保持简单且快速。
- **如果在一个事务中的命令出现错误，那么所有的命令都不会执行**；
- **如果在一个事务中出现运行错误，那么正确的命令会被执行**。

单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。

> https://zhuanlan.zhihu.com/p/43897838

Redis 的事务总是具有 ACID 中的一致性和隔离性，其他特性是不支持的。当服务器运行在 AOF 持久化模式下，并且 appendfsync 选项的值为 always 时，事务也具有持久性。

**事务命令：**

**MULTI：** 用于开启一个事务，它总是返回OK。MULTI执行之后，客户端可以继续向服务器发送任意多条命令，这些命令不会立即被执行，而是被放到一个队列中，当EXEC命令被调用时，所有队列中的命令才会被执行。

**EXEC：** 执行所有事务块内的命令。返回事务块内所有命令的返回值，按命令执行的先后顺序排列。当操作被打断时，返回空值 nil 。

**WATCH ：** 是一个乐观锁，可以为 Redis 事务提供 check-and-set （CAS）行为。可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行，监控一直持续到EXEC命令。

**DISCARD：** 调用该命令，客户端可以清空事务队列，并放弃执行事务，且客户端会从事务状态中退出。

**UNWATCH：** 命令可以取消watch对所有key的监控。

# 缓存

> [缓存雪崩、击穿、穿透](https://mp.weixin.qq.com/s?__biz=MzUxODAzNDg4NQ==&mid=2247490008&idx=1&sn=8f576e69ec63e02a8b42a00ae6754f0a&chksm=f98e5d72cef9d464710c891c4c0537c20e4949b39ee70c97c44c3f6f95df83fc406f52fc161b&mpshare=1&scene=24&srcid=0323a2FxUwktfSTrhxVg8ZpZ&sharer_sharetime=1616429322687&sharer_shareid=b6cb64bedb11e9fa59d690ebb6387d93&key=190f4c3d2818b8096a6e85b83bf894a12595c68420ef57586d68e1fd90e4b1a31e21a1c0ae715b4902ddb69c831d7f6902db6efc2209eb07aed322a0fdd73276214cdfec81a2ac1b8af89732de31b20afff19beecb239b72d06cdef7c8ba63c2cd8d99c2ce3f560b5a5f351ad0cb342dd46dcaf6667455d5e2e5ecc77d222001&ascene=14&uin=MTA2NzM1MDAzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=AadQNX2arcahSiXBOW5Jd2U%3D&pass_ticket=TH75Is630WzYlopDgihuWVrpncyi4%2Ftbhsug%2BQ4TehbcEeoaZG7BjLjiFyAvS%2Brx&wx_header=0)

## 缓存数据的处理流程是怎样的？

<img src="https://cdn.jsdelivr.net/gh/Simpleforever/imgbed/pic2/20210413144730.png" style="zoom:67%;" />

简单来说就是:

1. 如果用户请求的数据在缓存中就直接返回。
2. 缓存中不存在的话就看数据库中是否存在。
3. 数据库中存在的话就更新缓存中的数据并且返回。
4. 数据库中不存在的话就返回空数据。

## 分布式缓存和本地缓存有啥区别？让你自己设计本地缓存怎么设计？如何解决缓存过期问题？如何解决内存溢出问题？

分布式缓存一致性更好一点，用于集群环境下多节点使用同一份缓存的情况；有网络IO，吞吐率与缓存的数据大小有较大关系；

本地缓存非常高效，本地缓存会占用堆内存，影响垃圾回收、影响系统性能。

**本地缓存设计：**

以 Java 为例，使用自带的 map 或者 guava 实现的是本地缓存，最主要的特点是轻量以及快速，生命周期随着 jvm 的销毁而结束，并且在多实例的情况，每个实例都需要各自保存一份缓存，缓存不具有一致性。

**解决缓存过期：**

- 将缓存过期时间调为永久
- 将缓存失效时间分散开，不要将缓存时间长度都设置成一样；比如我们可以在原有的失效时间基础上增加一个随机值，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

**解决内存溢出：**

**第一步**，修改JVM启动参数，直接增加内存。(-Xms，-Xmx参数一定不要忘记加。)

**第二步**，检查错误日志，查看“OutOfMemory”错误前是否有其它异常或错误。

**第三步**，对代码进行走查和分析，找出可能发生内存溢出的位置。

## 缓存雪崩

指缓存同一时间大面积的失效，所以，后面的请求都会落到数据库上，造成数据库短时间内承受大量请求而崩掉。

举个例子：系统的缓存模块出了问题比如宕机导致不可用。造成系统的所有访问，都要走数据库。

还有一种缓存雪崩的场景是：**有一些被大量访问数据（热点缓存）在某一时刻大面积失效，导致对应的请求直接落到了数据库上。** 

**解决方案：**

- 针对 Redis 服务不可用的情况
  - 采用 Redis 集群，避免单机出现问题整个缓存服务都没办法使用
  - 限流，避免同时处理大量的请求
- 针对热点缓存失效的情况
  - 设置不同的失效时间，比如随机设置缓存的失效时间，防止同一时间大量数据过期现象发生
  - 一般并发量不是特别多的时候，使用最多的解决方案是加锁排队。
  - 给每一个缓存数据增加相应的缓存标记，记录缓存是否失效，如果缓存标记失效，则更新数据缓存。

## 缓存穿透

请求查询一条数据库中根本就不存在的数据，也就是缓存和数据库都查询不到这条数据，但是请求每次都会打到数据库上面去。说简单点就是大量请求的 key 根本不存在于缓存中，导致请求直接到了数据库上，根本没有经过缓存这一层。举个例子：某个黑客故意制造我们缓存中不存在的 key 发起大量请求，导致大量请求落到数据库。

**解决方案：**

- 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；

- **缓存无效 key**。从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击；

- 采用布隆过滤器，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力。

  具体是这样做的：把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话才会走下面的流程。

  但是，需要注意的是布隆过滤器可能会存在误判的情况。总结来说就是： **布隆过滤器说某个元素存在，小概率会误判。布隆过滤器说某个元素不在，那么这个元素一定不在。**

  > [《不了解布隆过滤器？一文给你整的明明白白！》](https://github.com/Snailclimb/JavaGuide/blob/master/docs/dataStructures-algorithms/data-structure/bloom-filter.md)

## 缓存击穿

**缓存击穿**是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据(过期了)，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。和缓存雪崩不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库

**解决方案：**

- 设置热点数据永远不过期
- 加互斥锁，互斥锁：​对缓存查询加锁，如果KEY不存在，就加锁，然后查DB入缓存，然后解锁，其他进程如果发现有锁就等待，然后等解锁后返回数据或者进入DB查询

## 如何保证缓存和数据库数据的一致性？

**方式一：**

读请求和写请求串行化，串到一个内存队列里去，这样就可以保证一定不会出现不一致的情况。串行化之后，就会导致系统的吞吐量会大幅度的降低，用比正常情况下多几倍的机器去支撑线上的一个请求。

**方式二：**

先更新数据库，假如读缓存失败，先读数据库，再回写缓存的方式实现

# redis数据分布方式？有什么优点？一致性hash呢？

## Hash

客户端分片：哈希+取余

节点伸缩：数据节点关系变化，导致数据迁移

迁移数量和添加节点数量有关：建议翻倍扩容

一个简单直观的想法是直接用Hash来计算，以Key做哈希后对节点数取模。可以看出，在key足够分散的情况下，均匀性可以获得，但一旦有节点加入或退出，所有的原有节点都会受到影响，稳定性无从谈起。

**服务器挂了会有缓存雪崩的情况发生**。

## 一致性Hash

客户端分片：哈希+顺时针（优化取余）

节点伸缩：只影响邻近节点，但是还是有数据迁移

翻倍伸缩：保证最小迁移数据和负载均衡

一致性Hash可以很好的解决稳定问题，可以将所有的存储节点排列在收尾相接的Hash环上，每个key在计算Hash后会顺时针找到先遇到的一组存储节点存放。而当有节点加入或退出时，仅影响该节点在Hash环上顺时针相邻的后续节点，将数据从该节点接收或者给予。但这有带来均匀性的问题，即使可以将存储节点等距排列，也会在**存储节点个数变化时带来数据的不均匀**。而这种可能成倍数的不均匀在实际工程中是不可接受的。


**偏移问题**：服务器在一堆了，使用虚拟结点

# redis主从复制

**主从复制原理：**

当启动一个 slave node 的时候，它会发送一个 PSYNC 命令给 master node。

如果这是 slave node 初次连接到 master node，那么会触发一次 full resynchronization 全量复制。此时 master 会启动一个后台线程，开始生成一份 RDB 快照文件，

同时还会将从客户端 client 新收到的所有写命令缓存在内存中。RDB 文件生成完毕后， master 会将这个 RDB 发送给 slave，slave 会先写入本地磁盘，然后再从本地磁盘加载到内存中，

接着 master 会将内存中缓存的写命令发送到 slave，slave 也会同步这些数据。

slave node 如果跟 master node 有网络故障，断开了连接，会自动重连，连接之后 master node 仅会复制给 slave 部分缺少的数据。

**过程原理**：

- 当从库和主库建立MS关系后，会向主数据库发送SYNC命令
- 主库接收到SYNC命令后会开始在后台保存快照(RDB持久化过程)，并将期间接收到的写命令缓存起来
- 当快照完成后，主Redis会将快照文件和所有缓存的写命令发送给从Redis
- 从Redis接收到后，会载入快照文件并且执行收到的缓存的命令
- 之后，主Redis每当接收到写命令时就会将命令发送从Redis，从而保证数据的一致

**缺点**：

所有的slave节点数据的复制和同步都由master节点来处理，会照成master节点压力太大，使用主从从结构来解决。

# 相关阅读

> [Redis的数据同步机制](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMTI2Ng==&mid=2247484167&idx=1&sn=6fcfb62ca3499791c8be1f01de16ebbb&chksm=e9d0cd4adea7445c40921f83445df8250d31af96971eb75b2cdb865abd951eaef551db0f01f1&scene=0&xtrack=1&key=544cb1a69a3b61ce6ade1a7b45cfabaeb206493215c02558c1f8a2e9d0524bf014b7dedcd287dee3aec7386f06ec81774e8fca3680d5c913bd0f044b5c5115e3557ad931f70147c7845e8cbd08f135181fd2b1eb081cc8ac83159a51c0ba2bb2cba4729bfe0bcfb9ea6d3b081ccb26584e089bdc9810f923632854028b28ccee&ascene=14&uin=MTA2NzM1MDAzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=AWfvmT18EixmQ6Uh%2Bg3E%2FHE%3D&pass_ticket=36BRpqMNJifb34ESQBgqBJSQWrtLUahK1TBlGqaasUB3Jiudbkafjx6qOwvSMlMX&wx_header=0)
>
> [3w字深度好文|Redis面试全攻略，读完这个就可以和面试官大战几个回合了](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMTI2Ng==&mid=2247484439&idx=1&sn=2b1199ccb150c99b4efea45e2a5f49d5&chksm=e9d0ca5adea7434cb5f525a53fe258180fe70a48c649cc2939bcc5a71d9e786c224a84ab5558&scene=0&xtrack=1&key=349f5f7a80de3dfc8fe24862b355333ca1a919ff313170ee7c5d511af9bb7f16c2d09b0627bd3bb4ce37ee67a5fa2058a90ea95b64d5f5088797df2f40a05e00d16335ec700de7b61c5ba2821909b49a6ae7afb599e6c65a03b6a045d5f59bf680fd641ad5cd3919e533c279b8152bd55e5b6bf5400c254be1b86bab48bb795d&ascene=14&uin=MTA2NzM1MDAzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=AQAJwu9GpYfenwZwiCSgyhU%3D&pass_ticket=36BRpqMNJifb34ESQBgqBJSQWrtLUahK1TBlGqaasUB3Jiudbkafjx6qOwvSMlMX&wx_header=0)
>
> [4W字的Redis面试教程](https://mp.weixin.qq.com/s?__biz=MzU1MzE4OTU0OQ==&mid=2247486139&idx=1&sn=52a021bcecc12f792830cb9888310d0d&chksm=fbf7e379cc806a6ffba968bcb3eebc242452f9f607ea3f27a2a43978043ce5898c048ea4ea0d&mpshare=1&scene=24&srcid=0904iIpuulljRagBwbybKEbx&sharer_sharetime=1599176439859&sharer_shareid=b6cb64bedb11e9fa59d690ebb6387d93&key=190f4c3d2818b809252ec0f3b62acbe1bdf32bfca5e68c95f5cb4689a82dcb14f149f60e9b249ef87ed71eaa0026f147851b53598bbd0636a0a9be2f8795ead5c4ecc8ab6b3bda6119af51d78e5e73843b69aa2cf94ef1c7f5702e5aca7ea0472b618116ece9b72d4b0ab11d5678d374b5257d9a6c77a720f3f47ae497674d16&ascene=14&uin=MTA2NzM1MDAzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=AcGA9dPqPwLvLEkUXVrLPWQ%3D&pass_ticket=TH75Is630WzYlopDgihuWVrpncyi4%2Ftbhsug%2BQ4TehbcEeoaZG7BjLjiFyAvS%2Brx&wx_header=0)
>
> [Redis 核心篇：唯快不破的秘密](https://mp.weixin.qq.com/s?__biz=Mzg3NTU3OTgxOA==&mid=2247489219&idx=1&sn=ca5b989074943a9d79e5b837abab579e&chksm=cf3e0606f8498f10a430198fe7ba88ec5842044443e1c1c09a5e2cccbb9fcd2daec8c3124ae9&mpshare=1&scene=24&srcid=0329rnoLcej3NkVOWbnehBN2&sharer_sharetime=1616978283907&sharer_shareid=b6cb64bedb11e9fa59d690ebb6387d93&key=79b605cf33a459ebf0510cc5ac789085e27c6ad5e81356e69096971823d9361f342321a4e3d2d7266ae72594d7988b85145b229b2b91d7756edd3f5d26653ba37d54a40f0248e7f72534fc323971c3164530a85a4765711dad95053d4c1e3d4ee65d56a2c22b174bc2b1c23c6fd0089dfee99b23c3bcda89177a4aa8aaf375a1&ascene=14&uin=MTA2NzM1MDAzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=AdbW0h84WceBZAekBC8y3LE%3D&pass_ticket=TH75Is630WzYlopDgihuWVrpncyi4%2Ftbhsug%2BQ4TehbcEeoaZG7BjLjiFyAvS%2Brx&wx_header=0)
>
> [Redis最佳实践](https://mp.weixin.qq.com/s?__biz=MzU1NTA0NTEwMg==&mid=2247487027&idx=1&sn=849a1489d267b20cc507dbd3a2dd9c52&chksm=fbdb17b2ccac9ea4b5438ad962ff60a89eb39ccb9d85215387204f4026caa5d06da2b774323a&mpshare=1&scene=24&srcid=040364SPTDjQ2s2yjQfDpOr5&sharer_sharetime=1617460084306&sharer_shareid=b6cb64bedb11e9fa59d690ebb6387d93&key=589f8c3d191a340843077c4c276d35563c6e158b75aeb0871b78416578a41b64b525ac151ea04c226442ef4187f94974b774f15cd975bbed3e8f950c31ca1d79138a3bd0f8248153eb2e95771be1967a28398e195d0ee195b2e9dc6f712e705b589d808d201e247b5036945fa12c55d2ac1fa77fc1e46c4a223764ddb0526f22&ascene=14&uin=MTA2NzM1MDAzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=ATTmum6UxePXYAr3NYogeZQ%3D&pass_ticket=TH75Is630WzYlopDgihuWVrpncyi4%2Ftbhsug%2BQ4TehbcEeoaZG7BjLjiFyAvS%2Brx&wx_header=0)
>
> [Redis 集群篇](https://mp.weixin.qq.com/s?__biz=MzIyNDI3MjY0NQ==&mid=2247488203&idx=1&sn=b8bea86017a9aecf58dea3ecbb512b67&chksm=e810daa1df6753b730a145f7985c8b9087cf28ca92ccb6ca0fa16b1a41a5a7e24f8c1629245f&mpshare=1&scene=24&srcid=04064c5wd5osfG1DNszWhTwt&sharer_sharetime=1617704813347&sharer_shareid=b6cb64bedb11e9fa59d690ebb6387d93&key=76c4ead5cdab479b6e48552567e82cca62aac14ea301383d90db65839e29d135e966f67d363e2dcae2626e0d2484971f4de62b2c5755840ec3e2c562e7469719ac4199e168dc0436c324289e6bf7806c0a411325f0c7fd32bae83061973093c7ee8205f0583eaa52db5641fdff37582fbfd284e5fea0a44195de6e86655723c0&ascene=14&uin=MTA2NzM1MDAzOA%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=AWbgF5bnaDtp%2BRXac0NrP0Q%3D&pass_ticket=TH75Is630WzYlopDgihuWVrpncyi4%2Ftbhsug%2BQ4TehbcEeoaZG7BjLjiFyAvS%2Brx&wx_header=0)