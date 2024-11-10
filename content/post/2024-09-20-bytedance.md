---
title:       "字节面经系列"
subtitle:    ""
date:        2024-09-20
author:      ""
image:       ""
tags:        ["resume"]
categories:  ["Tech" ]
---
## 字节面经

- https://www.kancloud.cn/leonshi/programming/2577007


## golang知识点
- [x] GMP调度模型, [详情](https://learnku.com/articles/41728)
- [ ] Context用法
- [x] 垃圾回收，优点和缺点, [详情](https://www.yuque.com/aceld/golang/zhzanb)
- [x] go语言逃逸分析，何时堆，何时栈？[详情](https://www.yuque.com/aceld/golang/yyrlis)
- [x] channel, [详情]()
- [x] defer的知识点，[详情](https://www.yuque.com/aceld/golang/qnubsg)
- [ ] gomodule, 学会基础用法即可, [详情](https://www.yuque.com/aceld/golang/wo6b5p)
- [x] go内存, [详情](https://www.yuque.com/aceld/golang/qzyivn)
- [ ] golang调试, [详情](https://debug-lixiwen.github.io/2021/07/18/shi-zhan/)
- [ ] 火焰图, [详情](https://bbs.huaweicloud.com/blogs/289818)
- [x] go中各模块的初始化顺序，[详情](https://segmentfault.com/a/1190000044323034)，[参考1](https://cloud.tencent.com/developer/article/2138066)，[参考2](https://studygolang.com/articles/6464)
- [x] go的逃逸分析，[详情]()
- [x] go的内存泄漏分析，[详情1](https://blog.csdn.net/caspar_notes/article/details/133224049?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-133224049-blog-128903295.235%5Ev43%5Econtrol&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-133224049-blog-128903295.235%5Ev43%5Econtrol&utm_relevant_index=5)，[详情2]()

### defer
- 先进后出
- 在return后执行（最终返回值要执行完defer再返回）
- defer在panic后依然有效果，所以可以用于捕获panic，但是只有最后一个panic可以被捕捉到
### channel
- 给nil的channel发送，永远阻塞
- 从nil的channel接收，永远阻塞
- 给已经关闭的channel发送，panic
- 从已关闭的channel接受，如果是缓冲区为空，收到0值
- 无缓冲的channel是同步的，有缓冲的channel是非同步的
- 15字口诀：“空读写阻塞，写关闭异常，读关闭空零”
### go的逃逸分析
1、go语言编译器会自动决定把一个变量放在栈还是放在堆，编译器会做逃逸分析(escape analysis)，当发现变量的作用域没有跑出函数范围，就可以在栈上，反之则必须分配在堆。
2、一般我们给一个引用类对象中的引用类成员进行赋值，可能出现逃逸现象。可以理解为访问一个引用对象实际上底层就是通过一个指针来间接的访问了，但如果再访问里面的引用成员就会有第二次间接访问，这样操作这部分对象的话，极大可能会出现逃逸的现象。
- []interface{}类型，通过[]赋值，值会逃逸
- map[string]interface{}类型，通过[]赋值，值会逃逸
- map[interface{}]interface{}，通过[]赋值，key和value都会逃逸
- map[string][]string, 赋值会发生[]string逃逸，
- []*int数据类型，赋值的右值会发生逃逸现象。
- func(*int)函数类型，进行函数赋值，会使传递的形参出现逃逸现象。
- func([]string): 函数类型，进行[]string{"value"}赋值，会使传递的参数出现逃逸现象。
- chan []string数据类型，想当前channel中传输[]string{"value"}会发生逃逸现象。
### go的内存泄漏
- goroutine导致的内存泄漏, 因为goroutine中阻塞的channel数据未被使用，使用方就已经退出了
- 互斥锁未被释放，产生的阻塞
- 资源链接未关闭，goroutine不会推出，例如http的body
- sync.waitGroup使用不当导致的阻塞
- 无限循环
- time.Ticker，需要close关闭
- 大字符串的截取，从而引发的暂时性的内存泄漏
- 大切片同理

## 操作系统&网络
- [ ] select和epoll, [详情](https://www.yuque.com/aceld/golang/sdgfgu)，
    一个阻塞，一个唤醒，唤醒还附带socket的提醒
- [ ] http和https，https的加密过程，需要几次通信
    tls需要4次握手
- [ ] tcp拥塞控制
- [ ] tcp三次握手和四次挥手，time_wait问题
    3次握手，4次，time_wait需要进行
- [ ] 自旋锁和互斥锁的区别
- [ ] 用过哪些分布式锁，优缺点

- [ ] 打开一个URL的过程
    DNS解析，HTTPS握手，
- [ ] 进程的调度
- [ ] CDN的工作原理，怎么就近访问，[详情](https://mp.weixin.qq.com/s/Pp0C8ALUXsmYCUkM5QnkQw)
    使用GSLB，1、基于Local DNS的静态调度；2、基于RTT（Round-Trip Time）的调度；3、基于成本和带宽的调度；4、基于服务等级的调度
    中心层、区域层、边缘层
- [x] quic协议详解，[详情](https://cloud.tencent.com/developer/article/1407615)，[详情1](https://www.51cto.com/article/704384.html)，[]
    基于UDP
- [x] 零拷贝技术，[详情](https://www.cnblogs.com/xiaolincoding/p/13719610.html)，[详情1](https://juejin.cn/post/6995519558475841550)，[详情2](https://github.com/0voice/linux_kernel_wiki/blob/main/%E6%96%87%E7%AB%A0/%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86/%E4%B8%80%E6%96%87%E5%B8%A6%E4%BD%A0%EF%BC%8C%E5%BD%BB%E5%BA%95%E4%BA%86%E8%A7%A3%EF%BC%8C%E9%9B%B6%E6%8B%B7%E8%B4%9DZero-Copy%E6%8A%80%E6%9C%AF.md)
    1、mmap+write
    2、sendfile
    3、sendfile+DMA收集
    4、splice
- [x] 守护进程，[详情](https://www.ruanyifeng.com/blog/2016/02/linux-daemon.html)
    1、Systemd
    2、nohup+&
    3、代码，fork+忽略各种信号，[详情](https://www.cnblogs.com/mickole/p/3188321.html)
- [x] 虚拟内存的作用，[详情](https://xiaolincoding.com/os/3_memory/vmem.html)
    第一，虚拟内存可以使得进程对运行内存超过物理内存大小，因为程序运行符合局部性原理，CPU 访问内存会有很明显的重复访问的倾向性，对于那些没有被经常使用到的内存，我们可以把它换出到物理内存之外，比如硬盘上的 swap 区域。
    第二，由于每个进程都有自己的页表，所以每个进程的虚拟内存空间就是相互独立的。进程也没有办法访问其他进程的页表，所以这些页表是私有的，这就解决了多进程之间地址冲突的问题。
    第三，页表里的页表项中除了物理地址之外，还有一些标记属性的比特，比如控制一个页的读写权限，标记该页是否存在等。在内存访问方面，操作系统提供了更好的安全性。

## 数据库
- [x] mysql数据结构，索引和B+树, [详情](https://cloud.tencent.com/developer/article/2292281)
- [x] 数据库特性ACID, [详情](https://www.yuque.com/aceld/golang/ycp0nb)
- [x] mysql幻读，如何避免
    实体化冲突--人为引入可以加锁的对象，我
    mvcc+next-key lock(间隙锁)
- [ ] 大库DDL
- [x] B树和B+树的区别，为什么 mysql 要用 B+ 树，mongodb 要用 B 树。[详情](https://juejin.cn/post/6844904136576598029)
- [ ] mysql如何保证数据的一致性，ACID
- [x] 如何说明innodb中B+树的层高，[详情](https://cloud.tencent.com/developer/article/2142040)
- [x] mysql中一条sql语句是怎么执行的，[详情](https://xiaolincoding.com/mysql/base/how_select.html#mysql-%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B%E6%98%AF%E6%80%8E%E6%A0%B7%E7%9A%84)
    1、连接器
    2、缓存
    3、解析器，词法分析、语法分析
    4、预处理器（语句是否合法，是否有相应字段）、优化器、执行器
- [x] mysql中count语句的效率对比，[详情]()
    count 1 count * 效率最高，有二级索引时会优先使用，没有则使用
    count主键次之，需要检查字段是否有值
    count字段
- [x] mysql中索引失效的情况，[详情](https://xiaolincoding.com/mysql/index/index_lose.html) 
    1、使用左/左右模糊匹配，即like %xx 和like %xx%
    2、对索引使用函数
    3、对索引进行表达式计算
    4、对索引隐式类型转换
    5、联合索引非最左匹配
    6、where 子句中用了 or
    7、走了错误的索引
- [x] mysql mvcc，[详情](https://www.jianshu.com/p/a21d0543503a), [详情1](https://blog.csdn.net/u011066470/article/details/133383277)
    row_id, trx_id, roll_pointer
- [x] mysql next-key-lock [详情](https://www.cnblogs.com/zhoujinyi/p/3435982.html)

### B+tree和B-tree的区别
B+tree只有叶子结点存储数据，树高度更低，局部性更好

## 中间件
- [ ] kafka原理, [详情](https://cloud.tencent.com/developer/article/1775065)
    使用，统一分发渠道，省略对接成本；异步限流控制，任务会有放大的效果，同时预防接口失败（处理是死循环），保留任务；异步限制爬取入库流量，处理速率固定
    如何保证一定传达，acks=-1/all，保证所有ISR副本都接收到了信息
- [ ] redis常用法, [详情]()
- [ ] redis setnx + expire 有什么缺点，如何优化。[详情](content/post/2024-08-29-redis-interview.md)
- [ ] redis 的跳表知道吗，为什么不用红黑树。[详情]()

### redis
- [x] LPOP和RPUSH的时间复杂度，双向链表是O(1)，压缩列表是O(N)，最坏O(N2)
- [ ] redis的基本数据类型
    string
    list，双向链表、压缩列表、quicklist
    hash，压缩列表、哈希表
    set，哈希表、整数集合


## 算法
- [x] 反转链表，穿针引线
- [x] 三数之和，注意排序和不要重复
- [x] 链表求和
- [x] 最小覆盖子串，比较复杂，需要看题解
- [ ] 最大交换
- [ ] 把一个方程式设计成树
- [x] 字符串处理（深信服）
- [x] 两个列表取取一样的值（富途）
- [x] 设计一个学生高考分数的存取api，1m的量
- [x] 找出列表中，比前面所有值大，以及比后面所有值小的数

## 架构
设计一个海量的评论系统
设计一个微信朋友圈系统，列出主要的表结构，只需要实现一些基础的功能
类似于设计一个长链接转短链接，不过需要考虑高并发，回答了分库分表。
设计一个海量日志写入系统


## 阿里的问题
- [x] 2个文件，1亿行字符串，如何找到相同的字符串，[海量数据相关](https://juejin.cn/post/6844904003998842887)
    分治，先用hash划分范围两个文件中的数据到同一个范围，然后map或者字典树解决
- [x] 介绍你主导的一个业务
    CDN成本结算
    增量更新
- [x] 如何用mysql实现乐观锁和悲观锁
    悲观：select for update
    乐观：判断版本是否对，然后更新/失败
- [x] mysql的隔离级别和问题
    读未提交--脏读，读到未提交的数据（可能回滚的数据）
    读已提交--不可重复读，在同一事务中，多次读取同一数据集合时，读到因为其他事务并发修改的结果
    可重复度--幻读，多次读时，因为新增加的行导致的问题--mvcc、next-key lock
    可串行化--超时、锁增用的问题
- [x] redis实现分布式锁，[详情](https://www.cnblogs.com/xiaolincoding/p/16517673.html)
    设置过期时间，SET key value NX PX time_millisecond
    解锁，DEL
    线程独立标识，只能自己解锁自己
    自动续期时间
