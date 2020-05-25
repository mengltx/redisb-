### 字典的实现

字典使用哈希表作为底层实现,一个哈希表中可以有多个哈希表节点,而每个哈希表节点保存了字典中的一个键值对.

#### 哈希表

redis字典中使用的哈希表由dict.h/dictht结构定义

```c
	typedef struct dictht{
    //哈希表数组
    dictEntry **table;
    
    //哈希表大小
    unsigned long size;
    
    //哈希表大小掩码,用于计算索引值,总是等于size-1
    unsigned long sizemask;
    
    //哈希表已有节点的数量
    unsigned long used;
    
  } dictht;
```



table属性是一个数组数组中每个元素都是指向dict.h/dictEntry结构的指针,每个dictEntry结构保存这一个键值对,size记录了哈希表的大小,即table数组的大小.used属性记录着哈希表已有节点(键值对)的数量.sizemask属性的值总是等于size-1,这个属性和哈希值一起决定一个键被放到table数组的哪个索引上.

<img src="/Users/zhangdongdong/Library/Application Support/typora-user-images/image-20200524212732096.png" alt="image-20200524212732096" style="zoom:50%;" />



#### 哈希表节点

哈希表节点使用dictEntry结构表示,每个dictEntry结构都保存着一个键值对:

```c
	typedef struct dictEntry{
    //键
    void * key;
    
    //值
    union{
      void *val;
      uint64_t u64;
      int64_t s64;
    } v;
    
    //指向下个哈希表节点,形成链表
    struct dictEntry *next;
  }dictEntry;
```



key属性保存着键值对中的键,而v属性保存着键值对中的值,其中值可以是一个指针,或者是一个uint64_t整数,又或者是一个int64_t整数.next属性是指向另一个哈希表节点的指针,这个指针将多个哈希值相同的键值对连接在一起,解决键冲突的问题.

<img src="/Users/zhangdongdong/Library/Application Support/typora-user-images/image-20200524213821230.png" alt="image-20200524213821230" style="zoom:50%;" />

#### 字典

Redis中字典由dict.h/dict结构表示

```c
typedef struct dict{
  //类型特定函数
  dictType *type;
  
  //私有数据
  void *privdata;
 
  //哈希表
  dictht ht[2];
  
  //rehash索引,当rehash不在进行时,值为-1
  int rehashidx;//rehashing not in progress if rehashidx == -1
}dict;
```

type属性和privdata属性是针对不同类型的键值对为创建多态字典而设置的;

+ type指向一个dictType类型的指针,每个dictType保存了一簇用于操作指定类型的键值对的函数,redis会为用途不同的字典设置不同的类型特定函数.
+ 而privdata属性则保存了需要传给那些特定函数的可选参数

```c
typedef struct dictType{
	//计算哈希值的函数
  unsigned int (*hashFunction)(const void *key);
  
  //复制键的函数
  void *(*keyDup)(void *privdata,const void *key);
  
  //复制值的函数
  void *(*valDup)(void *privdata,const void *key);
  
  //对比键的函数
  void *(*keyCompare)(void *privdata,const void *key1,const void *key2);
  
  //销毁键的函数
  void *(*keyDestructor)(void *privdata,const void *key);
  
  //销毁值的函数
  void *(*valDestructor)(void *privdata,const void *key);
}dictType;
```



ht属性是一个包含两个项的数组,数组的每一项都是dictht哈希表,一般情况下,字典只使用ht[0]哈希表,ht[1]哈希表只会在对ht[0]哈希表进行rehash的时候使用.

另一个和rehash有关的属性就是rehashidx,它记录了rehash目前的进度,如果目前没有进行rehash,那么它的值是-1.

下图展示里一个普通状态下的字典

![image-20200524220221215](/Users/zhangdongdong/Library/Application Support/typora-user-images/image-20200524220221215.png)



### 哈希算法

要将一个新的键值对添加到字典里面时,程序需要先根据键值对的键计算出哈希值和索引值,然后再根据索引值.将包含新键值对的哈希表节点放到哈希表数组指定的索引上面

Redis计算哈希值和索引值的方法如下:

```
#使用字典设置的哈希函数,计算key的哈希值
hash = dict->type->hashFunction(key);

#使用哈希表的sizemask属性和哈希值计算索引值
#根据情况不同,ht[x]可以是ht[0]或者ht[1];
index  = hash & dict->ht[x].sizemask;

```

当字典被当做数据库的底层实现,或者哈希键的底层实现,Redis使用MurmurHash2算法来计算哈希值.



### 解决键的冲突

当两个或者两个以上的键被分配到哈希表数组的同一个索引上时,我们称这些键发生了冲突(collision)

Redis使用链地址法(separate chaining)来解决键冲突.每个哈希表节点都有一个next指针构成了一个单项链表,被分配到同一个索引上的多个节点可以使用单向链表连接起来,这样就解决了键冲突.

