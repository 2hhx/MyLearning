***innodb可重复读隔离级别解决国不可重复读在问题，利用MVCC版本解决的。***

***innode利用*****next-key locks间隙锁解决的是当前读情况下的幻读\*。***

***脏读和不可重复读都是一个事务读，另外一个事务写引起的。\***

***而两个事务一起写造成读写冲突时，造成丢失更新：一个事务的更新覆盖了其它事务的更新结果。解决的办法有：\***

- ***使用Serializable隔离级别，事务是串行执行的！***
- ***乐观锁***
- ***悲观锁***

 

***如果我们的锁用到索引就是行锁，如果没有用到索引就是表锁，但是我们操作的数据必须用到锁才行。***

***注意几点：\***

***1.行锁必须有索引才能实现，否则会自动锁全表，那么就不是行锁了。\***

***2.两个事务不能锁同一个索引\***

# InnoDB存储引擎

InnoDB存储引擎**支持事务**，其特点是**行锁设计、支持外键**，并支持类似于Oracle的**非锁定读**，即默认**读取操作不会产生锁**。从MySQL数据库5.5.8版本开始，InnoDB存储引擎是默认的存储引擎。

InnoDB通过使用**多版本并发控制（MVCC）来获得高并发性**，并且实现了SQL标准的**4种隔离级别**，默认为**REPEATABLE(可重复读，避免脏读、不可重复读)**级别。同时，使用一种被称为**next-key locking**的策略来**避免幻读**（phantom）现象的产生。

## 连接MySQL

连接MySQL操作是一个**连接进程和MySQL数据库实例进行通信**。本质上是**进程通信**。**常用的进程通信方式有管道、命名管道、命名字、TCP/IP套接字、UNIX域套接字。**

### TCP/IP套接字

TCP/IP套接字方式是MySQL数据库在任何平台下都提供的连接方式，也是**网络中使用得最多的一种方式**。这种方式**在TCP/IP连接上建立一个基于网络的连接请求**，一般情况下客户端（client）在一台服务器上，而MySQL实例（server）在另一台服务器上，这两台机器通过一个TCP/IP网络连接。

客户端是Windows，它向一台Host IP为192.168.0.101的MySQL实例**发起了TCP/IP连接请求**，并且连接成功。之后就可以对MySQL数据库进行一些数据库操作，如DDL和DML等。

### 命名管道和共享内存

如果两个需要进程通信的进程在同一台服务器上，那么可以使用命名管道

## InnoDB四大特性

- 插入缓冲（Insert Buffer）

- 两次写（Double Write）

- 自适应哈希索引（Adaptive Hash Index）：哈希（hash）是一种非常快的查找方法，在一般情况下这种查找的时间复杂度为O(1)，即一般仅需要一次查找就能定位数据。而B+树的查找次数，取决于B+树的高度，在生产环境中，B+树的高度一般为3～4层，故需要3～4次的查询。如果观察到建立哈希索引可以带来速度提升，则建立哈希索引，称之为自适应哈希索引（Adaptive Hash Index，AHI）。

- 预读（通过AIO实现）：InnoDB使用两种预读算法来提高I/O性能：**线性预读**（linear read-ahead）和**随机预读**（randomread-ahead）

## 重做日志文件（redo log file）

当实例或介质失败（media failure）时，重做日志文件就能派上用场。例如，数据库由于所在主机掉电导致实例失败，InnoDB存储引擎会使用重做日志**恢复到掉电前的时刻**，以此来保证**数据的完整性**。

## InnoDB索引

 InnoDB存储引擎支持以下几种常见的索引：

- B+树索引：
  - B+树索引就是传统意义上的索引，这是目前关系型数据库系统中查找最为常用和最为有效的索引。B+树索引的构造类似于二叉树，根据键值（Key Value）快速找到数据。
  - 注意　B+树中的B不是代表二叉（binary），而是代表平衡（balance），因为B+树是从最
    早的平衡二叉树演化而来，但是B+树不是一个二叉树。
  - B+树索引并不能找到一个给定键值的具体行。B+树索引能找到的只是被查找数据行所在的页。然后数据库通过把页读入到内存，再在内存中进行查找，最后得到要查找的数据。
- 全文索引：是目前搜索引擎使用的一种关键技术
- 哈希索引：InnoDB存储引擎支持的哈希索引是自适应的，InnoDB存储引擎会根据表的使用情况自动为表生成哈希索引，不能人为干预是否在一张表中生成哈希索引。

## B+树索引

B+树是通过二叉查找树，再由平衡二叉树，B树演化而来。

### 二叉查找树和平衡二叉树

#### 二叉查找树

在二叉查找树中，左子树的键值总是小于根的键值，右子树的键值总是大于根的键值。因此可以通过**中序遍历得到键值的排序输出**

#### 平衡二叉树（或称为AVL树）

平衡二叉树的定义如下：首先符合二叉查找树的定义，其次必须满足任何节点的两个子树的高度最大差为1。

平衡二叉树的查找性能是比较高的，但不是最高的，只是接近最高性能。最好的性能需要建立一棵最优二叉树，但是最优二叉树的建立和维护需要大量的操作(需要1次或多次左旋和右旋来得到插入或更新后树的平衡性)。

### B树

B树为**平衡多路查找树**，当其阶数为2时就是二叉平衡查找树。B树每个节点都存储key和data，所有节点组成这棵树，并且叶子节点指针为null。

![image-20200513102524814](C:\Users\wwl\Documents\学习\MyLearning\java\images\image-20200513102524814.png)

### B+树特征

B+ 树是一种树数据结构，是一个n叉树，每个节点通常有多个孩子，一颗B+树包含根节点、内部节点和叶子节点。B+ 树通常用于数据库和操作系统的文件系统中。 B+ 树的特点是能够

- **保持数据稳定有序**，
- **插入与修改拥有较稳定的对数时间复杂度**
-  B+ 树元素**自底向上插入**。

![image-20200513102642382](C:\Users\wwl\Documents\学习\MyLearning\java\images\image-20200513102642382.png)

B+树是为**磁盘**或其他直接存取辅助设备设计的一种**平衡查找树**。

在B+树中，所有记录节点都是按键值的大小顺序存放在同一层的叶子节点上，由各叶子节点指针进行连接。先来看一个B+树，其高度为2，每页可存放4条记录。

![image-20200513093158635](C:\Users\wwl\Documents\学习\MyLearning\java\images\image-20200513093158635.png)

所有记录都在叶子节点上，并且是顺序存放的，如果用户从最左边的叶子节点开始顺序遍历，可以得到所有键值的顺序排序：5、10、15、20、25、30、50、55、60、65、75、80、85、90。

### B+树的插入操作

![image-20200513093955886](C:\Users\wwl\Documents\学习\MyLearning\java\images\image-20200513093955886.png)

### B+树索引

B+树的高度一般都在2～4层，这也就是说查找某一键值的行记录时最多只需要2到4次IO

数据库中的B+树索引可以分为聚集索引（clustered inex）和辅助索引（secondary index），**聚集索引与辅助索引不同的是，叶子节点存放的是否是一整行的信息。**

#### 聚集索引

**聚集索引（clustered index）就是按照每张表的主键构造一棵B+树，同时叶子节点中存放的即为整张表的行记录数据，也将聚集索引的叶子节点称为数据页。**由于实际的数据页只能按照一棵B+树进行排序，因此每张表只能拥有一个聚集索引。

#### 辅助索引(非聚集索引)

对于辅助索引（Secondary Index，也称非聚集索引），叶子节点并**不包含行记录的全部数据**。 叶 子 节 点 除 了 包 含 **键 值** 以 外， 每 个 叶 子 节 点 中 的 索 引 行 中 还 包 含 了 一 个 **书 签（bookmark）**。该书签用来告诉InnoDB存储引擎哪里可以找到与索引相对应的行数据。由于InnoDB存储引擎表是**索引组织表**，因此InnoDB存储引擎的辅**助索引的书签就是相应行数据的聚集索引键**。

**举例来说**，如果在一棵高度为3的辅助索引树中查找数据，那需要对这棵辅助索引树遍历3次找到指定主键，如果聚集索引树的高度同样为3，那么还需要对聚集索引树进行3次查找，最终找到一个完整的行数据所在的页，因此一共需要6次逻辑IO访问以得到最终的一个数据页。

对于其他的一些数据库，如Microsoft SQL Server数据库，其有一种称为**堆表**的表类型，即行数据的存储按照插入的顺序存放。这与MySQL数据库的MyISAM存储引擎有些类似。堆表的特性决定了**堆表上的索引都是非聚集的**，主键与非主键的区别只是是否唯一且非空（NOTNULL）。因此**这时书签是一个行标识符（Row IdentifiedrRID）**，可以用如“文件号：页号：槽号”的格式来定位实际的行数据。

#### 覆盖索引

- 解释一： 就是select的数据列只用从索引中就能够取得，不必从数据表中读取，换句话说查询列要被所使用的索引覆盖。
- 解释二： 索引是高效找到行的一个方法，当能通过检索索引就可以读取想要的数据，那就不需要再到数据表中读取行了。如果一个索引包含了（或覆盖了）满足查询语句中字段与条件的数据就叫 做覆盖索引。
- 解释三：是非聚集组合索引的一种形式，它包括在查询里的Select、Join和Where子句用到的所有列（即建立索引的字段正好是覆盖查询语句[select子句]与查询条件[Where子句]中所涉及的字段，也即，索引包含了查询正在查找的所有数据）。

### 堆组织表(HOT)和索引组织表(IOT)的区别

   ① 堆组织表，在Heap Table中，数据行是按照**“随机存取”**的方式进行管理。Index叶子节点上记录的数据行**键值和Rowid取值**(**书 签（bookmark）**比如，文件号：页号：槽号)，查找的时候先找索引，然后再根据索引rowid找到块中的行数据。**索引和表数据是分离的**
   ② 索引组织表，**数据行的组织并不是随机的**，而是依据数据表主键，按照索引树进行保存**[物理存储并不一定有序，逻辑有序：叶子节点有一个双向链表来保证有序性]**。其**行数据以索引形式存放，因此找到索引，就等于找到了行数据**。**索引和数据是在一起的**

## 锁

锁是数据库系统区别于文件系统的一个关键特性。数据库系统使用锁是为了支持**对共享资源进行并发访问**，提供**数据的完整性和一致性。**

#### 表锁、页锁和行锁

对于**MyISAM**引擎，其锁是表锁设计。并发情况下的读没有问题，但是并发插入时的性能就要差一些了。

在Microsoft SQL Server **2005版本之前**其都是**页锁**的，相对表锁的MyISAM引擎来说，并发性能有所提高。页锁容易实现，然而对于热点数据页的并发问题依然无能为力。

到2005版本，Microsoft SQL Server开始支持乐观并发和悲观并发，在乐观并发下开始支持行级锁，在
**SQL Server下，锁是一种稀有的资源，锁越多开销就越大，因此它会有锁升级。**在这种情况下，**行锁会升级到表锁**，这时并发的性能又回到了以前。

**InnoDB存储引擎**提供一致性的**非锁定读**、**行级锁**支持。**行级锁没有相关额外的开销，所以没有锁升级**，并可以同时得到并发性和一致性。

#### Lock和latch

![image-20200513111841137](C:\Users\wwl\Documents\学习\MyLearning\java\images\image-20200513111841137.png)

### Lock锁的类型

#### 行锁

InnoDB存储引擎实现了如下两种标准的行级锁：

- 共享锁（S Lock），允许事务读一行数据。
- 排他锁（X Lock），允许事务删除或更新一行数据。
- X锁与任何锁都不兼容，只有S锁和S锁兼容。
- **S和X锁都是行锁，兼容是指对同一记录（row）锁的兼容性情况。**

**举例：**如果一个事务T1已经获得了行r的共享锁，那么另外的事务T2可以立即获得行r的共享锁，因为读取并没有改变行r的数据，称这种情况为锁兼容（Lock Compatible）。但若有其他的事务T3想获得行r的排他锁，则其必须等待事务T1、T2释放行r上的共享锁——这种情况称为锁不兼容。

![image-20200513112227507](C:\Users\wwl\Documents\学习\MyLearning\java\images\image-20200513112227507.png)

#### 意向锁

InnoDB 支持`多粒度锁（multiple granularity locking）`，它允许`行级锁`与`表级锁`共存，而**意向锁**就是其中的一种`表锁`。

意向锁是将锁定的对象分为多个层次，意向锁意味着事务希望在更细粒度（fine granularity）上进行加锁，如下图所示

![image-20200513112535086](C:\Users\wwl\Documents\学习\MyLearning\java\images\image-20200513112535086.png)

若将上锁的对象看成一棵树，那么对最下层的对象上锁，也就是**对最细粒度的对象进行上锁**，那么**首先需要对粗粒度的对象上锁。**

**如果需要对页上的记录r进行上X锁，那么分别需要对数据库A、表、页上意向锁IX，最后对记录r上X锁。若其中任何一个部分导致等待，那么该操作需要等待粗粒度锁的完成。举例**来说，在对记录r加X锁之前，已经有事务对表1进行了S表锁，那么表1上已存在S锁，之后事务需要对记录r在表1上加上IX，由于不兼容，所以该事务需要等待表锁操作的完成。

1）意向共享锁（IS Lock），事务想要获得一张表中某几行的共享锁
2）意向排他锁（IX Lock），事务想要获得一张表中某几行的排他锁

![image-20200513113514002](C:\Users\wwl\Documents\学习\MyLearning\java\images\image-20200513113514002.png)

**注意：这里的排他 / 共享锁指的都是表锁！！！意向锁不会与行级的共享 / 排他锁互斥！！！**

#### 总结

1. InnoDB 支持`多粒度锁`，特定场景下，行级锁可以与表级锁共存。
2. 意向锁之间互不排斥，但除了 IS 与 S 兼容外，`意向锁会与 共享锁 / 排他锁 互斥`。
3. IX，IS是表级锁，不会和行级的X，S锁发生冲突。只会和表级的X，S发生冲突。
4. 意向锁在保证并发性的前提下，实现了`行锁和表锁共存`且`满足事务隔离性`的要求。

### 一致性非锁定读

![image-20200513125519140](C:\Users\wwl\Documents\学习\MyLearning\java\images\image-20200513125519140.png)

图6-4直观地展现了InnoDB存储引擎一致性的非锁定读。之所以称其为非锁定读，因为不需要等待访问的行上X锁的释放。**快照数据是指该行的之前版本的数据**，该实现是通过undo段来完成。而undo用来在事务中回滚数据，因此快照数据本身是没有额外的开销。此外，读取快照数据是不需要上锁的，因为没有事务需要对历史的数据进行修改操作。

在InnoDB存储引擎的默认设置下，这是**默认的读取方式**，即读取不会占用和等待表上的锁。但是**在不同事务隔离级别下，读取的方式不同**，并不是在每个事务隔离级别下都是采用非锁定的一致性读。此外，即使都是使用非锁定的一致性读，但是对于快照数据的定义也各不相同。

### 一致性锁定读

InnoDB存储引擎对于SELECT语句支持两种一致性的锁定读（locking read）操作：

- SELECT…FOR UPDATE(加X锁)
- SELECT…LOCK IN SHARE MODE(加S锁)

SELECT…FOR UPDATE对读取的行记录加一个X锁，其他事务不能对已锁定的行加上任何锁。SELECT…LOCK IN SHARE MODE对读取的行记录加一个S锁，其他事务可以向被锁定的行加S锁，但是如果加X锁，则会被阻塞。对于一致性非锁定读，即使读取的行已被执行了SELECT…FOR UPDATE，也是可以进行读取的，

### 行锁的3种算法

**在默认的事务隔离级别下，即REPEATABLE READ下，InnoDB存储引擎采用Next-Key Locking机制来避免Phantom Problem（幻像问题）**

- Record Lock：单个行记录上的锁

- Gap Lock：:在索引记录之间的间隙中加锁，或者是在某一条索引记录之前或者之后加锁，并不包括该索引记录本身。gap lock的机制主要是解决可重复读模式下的幻读问题。

- Next-Key Lock∶Gap Lock+Record Lock，锁定一个范围，并且锁定记录本身：例如一个索引有10，11，13和20这四个值，那么该索引可能被Next-Key Locking的区间为：**（左开右毕）**

  然而，当**查询的索引含有唯一属性**时，InnoDB存储引擎会对Next-Key Lock进行优化，将其
  **降级为Record Lock**，即仅锁住索引本身，而不是范围。

  - (-∞,10]
  - (10,11]
  - (11,13]
  - (13，20]
  - (20,+∞)

### 多版本并发控制(MVCC)

通过图6-4可以知道，快照数据其实就是当前行数据之前的历史版本，每行记录可能有多个版本。就图6-4所显示的，一个行记录可能有不止一个快照数据，一般称这种技术为行多版本技术。由此带来的并发控制，称之为多版本并发控制（Multi Version Concurrency Control，MVCC）。

- 每开启一个新的事务，系统版本号都会自动递增。事务开始时刻的系统版本号会作为事务的版本号，用来和查询到的每行记录的版本号进行比较。

- 在READ COMMITTED事务隔离级别下，对于快照数据，非一致性读总是读取被锁定行的最新一份快照数据。（在一个事务内，多次读同一个数据。在这个事务还没有结束时，另一个事务也访问该同一数据并修改数据。那么，在第一个事务的两次读数据之间。由于另一个事务的修改，那么第一个事务两次读到的数据可能不一样，这样就发生了**在一个事务内两次读到的数据是不一样的**，**违反了数据库的隔离性，即当前事务能够看到其他事务的结果**，因此称为不可重复读，即**原始读取不是可重复的**。）
- 而在REPEATABLE READ事务隔离级别下，对于快照数据，非一致性读总是读取事务开始时的行数据版本。**一个事务中读取的快照都是一个，所以可以解决不可重复读的问题。**

#### undo log

 **undo log中记录的是数据表记录行的多个版本，也就是事务执行过程中的回滚段,其实就是MVCC 中的一行原始数据的多个版本镜像数据。**

![image-20200513125519140](C:\Users\wwl\Documents\学习\MyLearning\java\images\image-20200513125519140.png)

