+ 双端: 链表节点都带有prev和next指针,获取某个节点的前置节点和后置几点的时间复杂度都是o(1);
+ 无环:表头节点的prev和表尾节点的next都指向NULL,对链表的访问以NULL为终点;
+ 带表头指针和表尾指针:可快速获取表头节点和表尾节点,时间复杂度均为o(1);
+ 带链表的长度计数器:list结构带有len属性记录链表的长度
+ 多态:链表节点使用void*指针来保存节点值,并且可以通过list结构的dump,free,match三个属性为节点设置类型特定的函数,所以链表可以用于保存不同类型的值.



 ![dsfcds](https://tva1.sinaimg.cn/large/007S8ZIlgy1gf3leb25i2j30s50fvab1.jpg)

|                    |                                                              | 时间复杂度                                   |
| ------------------ | ------------------------------------------------------------ | -------------------------------------------- |
| listSetDupMethod   | 将给定的函数设置为链表的节点值复制函数                       | o(1)                                         |
| listGetDupMethod   | 返回链表当前使用的节点值复制函数                             | 通过链表的dup属性直接获得,o(1)               |
| listSetFreeMethod  | 将给定的函数设置为链表的节点值释放函数                       | o(1)                                         |
| listGetFree        | 返回链表当前使用的节点值释放函数                             | 通过链表的free属性直接获得,o(1)              |
| listSetMatchMethod | 将给定的函数设置为链表的节点值对比函数                       | o(1)                                         |
| listGetMatchMethod | 返回链表当前的节点值对比函数                                 | 对比函数可以通过链表的match属性直接获取,o(1) |
| listLength         | 返回链表的长度(包含多少个节点)                               | 可以通过链表的len属性直接获取,o(1)           |
| listFirst          | 返回链表表头节点                                             | 可以通过链表的head属性直接获取,o(1)          |
| listLast           | 返回链表的表尾节点                                           | 可以通过链表的tail属性直接获取,o(1)          |
| listPrevNode       | 返会给定节点的前置节点                                       | 可以通过节点的prev属性直接获取,o(1)          |
| listNextNode       | 返回给定节点的后置节点                                       | 可以通过节点的next属性直接获取,o(1)          |
| listNodeValue      | 返回给定节点目前保存的值                                     | 可以通过节点的value属性直接获取,o(1)         |
| listCreate         | 创建一个不包含节点的新链表                                   | o(1)                                         |
| listAddNodeHead    | 将一个包含给定值的新节点添加到链表的表头                     | o(1)                                         |
| listAddNodeTail    | 将一个包含给定值的新节点添加到链表的表头                     | o(1)                                         |
| listInsertNode     | 将一个包含给定值的新节点添加到给定节点之前或者之后           | o(1)                                         |
| listSearchKey      | 查找并返回包含给定值的节点                                   | o(N),N为链表长度                             |
| listIndex          | 返回链表给定索引上的节点                                     | o(N),N为链表长度                             |
| listDelNode        | 从链表中删除给定节点                                         | o(N),N为链表长度                             |
| listRotate         | 将链表的表尾节点弹出然后将被弹出的节点插入到链表的表头,成为新的表头节点 | o(1)                                         |
| listDup            | 复制一个给定链表的副本                                       | o(N),N为链表长度                             |
| listRelease        | 释放给定链表,以及链表中的节点                                | o(N),N为链表长度                             |
|                    |                                                              |                                              |

