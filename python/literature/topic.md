Global Interpreter Lock(全局解释器锁)
====================================
Python代码的执行由Python 虚拟机(也叫解释器主循环，CPython版本)来控制，Python 在设计之初就考虑到要在解释器的主循环中，同时
只有一个线程在执行，即在任意时刻，只有一个线程在解释器中运行。对Python 虚拟机的访问由全局解释器锁（GIL）来控制，正是这个锁能
保证同一时刻只有一个线程在运行。


    1. 在多线程环境中，Python 虚拟机按以下方式执行：
        1. 设置GIL
        2. 切换到一个线程去运行
        3. 运行：
            a. 指定数量的字节码指令
            b. 线程主动让出控制（可以调用time.sleep(0)）
        4. 把线程设置为睡眠状态
        5. 解锁GIL
        6. 再次重复以上所有步骤

    2. 在调用外部代码（如C/C++扩展函数）的时候，GIL 将会被锁定，直到这个函数结束为止（由于在这期间没有Python的字节码被
    运行，所以不会做线程切换）。
 
 
 迭代器和生成器的区别
 =================
 
    1. 迭代器是一个更抽象的概念，任何对象，如果它的类有next方法和iter方法返回自己本身。对于string、list、dict、tuple等
    这类容器对象，使用for循环遍历是很方便的。在后台for语句对容器对象调用iter()函数，iter()是python的内置函数。iter()会
    返回一个定义了next()方法的迭代器对象，它在容器中逐个访问容器内元素，next()也是python的内置函数。在没有后续元素时，
    next()会抛出一个StopIteration异常
    
    2. 生成器（Generator）是创建迭代器的简单而强大的工具。它们写起来就像是正规的函数，只是在需要返回数据的时候使用yield语
    句。每次next()被调用时，生成器会返回它脱离的位置（它记忆语句最后一次执行的位置和所有的数据值）

    3. 区别：生成器能做到迭代器能做的所有事,而且因为自动创建了__iter__()和next()方法,生成器显得特别简洁,而且生成器也是高
    效的，使用生成器表达式取代列表解析可以同时节省内存。除了创建和保存程序状态的自动方法,当发生器终结时,还会自动抛出
    StopIteration异常


Python内存管理简要说明
====================

* 垃圾回收：python不像C++，Java等语言一样，他们可以不用事先声明变量类型而直接对变量进行赋值。对Python语言来讲，对象的类型和内存都是在运行时确定的。这也是为什么我们称Python语言为动态类型的原因（这里我们把动态类型可以简单的归结为对变量内存地址的分配是在运行时自动判断变量类型并对变量进行赋值）。
* 引用计数：Python采用了类似Windows内核对象一样的方式来对内存进行管理。每一个对象，都维护这一个对指向该对对象的引用的计数。当变量被绑定在一个对象上的时候，该变量的引用计数就是1，(还有另外一些情况也会导致变量引用计数的增加),系统会自动维护这些标签，并定时扫描，当某标签的引用计数变为0的时候，该对就会被回收。
* 内存池机制Python的内存机制以金字塔行，-1，-2层主要有操作系统进行操作，
--------------
    第0层是C中的malloc，free等内存分配和释放函数进行操作；
    第1层和第2层是内存池，有Python的接口函数PyMem_Malloc函数实现，当对象小于256K时有该层直接分配内存；
    第3层是最上层，也就是我们对Python对象的直接操作；

* 在C中如果频繁的调用 malloc 与 free 时,是会产生性能问题的.再加上频繁的分配与释放小块的内存会产生内存碎片. Python 在这里主要干的工作有:  
-------------------

    如果请求分配的内存在1~256字节之间就使用自己的内存管理系统,否则直接使用 malloc.
    这里还是会调用 malloc 分配内存,但每次会分配一块大小为256k的大块内存.
    经由内存池登记的内存到最后还是会回收到内存池,并不会调用 C 的 free 释放掉.以便下次使用.对于简单的Python对象，
    例如数值、字符串，元组（tuple不允许被更改)采用的是复制的方式(深拷贝?)，也就是说当将另一个变量B赋值给变量A时，
    虽然A和B的内存空间仍然相同，但当A的值发生变化时，会重新给A分配空间，A和B的地址变得不再相同


MyISAM 与 InnoDB
================

    1. InnoDB 支持事务，MyISAM 不支持，这一点是非常之重要。事务是一种高级的处理方式，如在一些列增删改中只要哪个出错还可
    以回滚还原，而 MyISAM就不可以了；
    2. MyISAM 适合查询以及插入为主的应用，InnoDB 适合频繁修改以及涉及到安全性较高的应用；
    3. InnoDB 支持外键，MyISAM 不支持；
    4. MyISAM 是默认引擎，InnoDB 需要指定；
    5. InnoDB 不支持 FULLTEXT 类型的索引；
    6. InnoDB 中不保存表的行数，如 select count(*) from table 时，InnoDB；需要扫描一遍整个表来计算有多少行，但是 MyISAM
    只要简单的读出保存好的行数即可。注意的是，当 count(*)语句包含 where 条件时 MyISAM 也需要扫描整个表；
    7. 对于自增长的字段，InnoDB 中必须包含只有该字段的索引，但是在 MyISAM表中可以和其他字段一起建立联合索引；
    8. 清空整个表时，InnoDB 是一行一行的删除，效率非常慢。MyISAM 则会重建表；
    9. InnoDB 支持行锁（某些情况下还是锁整表，如 update table set a=1 whereuser like '%lee%'

    
* 数据库优化
---------

    1. 优化索引、SQL 语句、分析慢查询；
    2. 设计表的时候严格根据数据库的设计范式来设计数据库；
    3. 使用缓存，把经常访问到的数据而且不需要经常变化的数据放在缓存中，能节约磁盘IO；
    4. 优化硬件；采用SSD，使用磁盘队列技术(RAID0,RAID1,RDID5)等；
    5. 采用MySQL 内部自带的表分区技术，把数据分层不同的文件，能够提高磁盘的读取效率；
    6. 垂直分表；把一些不经常读的数据放在一张表里，节约磁盘I/O；
    7. 主从分离读写；采用主从复制把数据库的读操作和写入操作分离开来；
    8. 分库分表分机器（数据量特别大），主要的的原理就是数据路由；
    9. 选择合适的表引擎，参数上的优化；
    10. 进行架构级别的缓存，静态化和分布式；
    11. 不采用全文索引；
    12. 采用更快的存储方式，例如 NoSQL存储经常访问的数据
    cur = conn.cursor(cursor=pymysql.cursors.DictCursor)