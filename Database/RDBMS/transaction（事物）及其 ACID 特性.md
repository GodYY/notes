[TOC]



# 1.  事物简介

## 1.1 事物的定义

事务（Transaction）是由一系列对系统中数据进行访问或更新的操作所组成的一个程序执行逻辑单元（Unit）。在计算机术语中，事务通常就是指数据库事务 。

![one_transaction](../../resource/Database/RDBMS/one_transaction.png)

在数据库管理系统（DBMS）中，事务是数据库恢复和并发控制的基本单位。它是一个操作序列，这些操作要么都执行，要么都不执行，它是一个不可分割的工作单位。

例如，银行转帐工作：从源帐号扣款并使目标帐号增款，这两个操作必须要么全部执行，要么都不执行，否则就会出现该笔金额平白消失或出现的情况。所以，应该把他们看成一个事务。

在现代数据库中，事务还可以实现其他一些事情，例如，确保你不能访问别人写了一半的数据；但是基本思想是相同的——事务是用来确保**无论发生什么情况，你使用的数据都将处于一个合理的状态**：

> transactions are there to ensure, that no matter what happens, the data you work with will be in a sensible state.

它保证在任何情况下都不会出现在转账后从一个帐户中扣除了资金，而未将其存入另一个帐户的情况。

## 1.2 事物的目的

数据库事务通常包含了一个序列的对数据库的读/写操作。包含有以下两个目的：

1. 为数据库操作序列提供了一个从失败中恢复到正常状态的方法，同时提供了数据库即使在异常状态下仍能保持一致性的方法。
2. 当多个应用程序在并发访问数据库时，可以在这些应用程序之间提供一个隔离方法，以防止彼此的操作互相干扰。

当事务被提交给了DBMS，则DBMS需要确保该事务中的所有操作都成功完成且其结果被永久保存在数据库中，如果事务中有的操作没有成功完成，则事务中的所有操作都需要回滚，回到事务执行前的状态；同时，该事务对数据库或者其他事务的执行无影响，所有的事务都好像在独立的运行。

Martin Kleppmann在他的《Designing Data-Intensive Applications》一书中有提到：

> Transactions are not a law of nature; they were created with a purpose, namely to simplify the programming model for applications accessing a database. By using transactions, the application is free to ignore certain potential error scenarios and concurrency issues, because the database takes care of them instead (we call these safety guarantees).

在现实情况下，失败的风险很高。在一个数据库事务的执行过程中，有可能会遇上事务操作失败、数据库系统或操作系统出错，甚至是存储介质出错等情况。而上述Martin的话说明了事务的存在，就是为了能够简化我们的编程模型，不需要我们去考虑各种各样的潜在错误和并发问题 。我们在实际使用事务时，不需要考虑数据库宕机，网络异常，并发修改等问题，整个事务要么提交，要么回滚，非常方便。所以本质上来说，**事务的出现了是为了应用层服务的，而不是数据库系统本身的需要** 。

## 1.3 事务的状态

因为事务具有原子性，所以从外部看的话，事务就是密不可分的一个整体，事务的状态也只有三种：Active、Commited 和 Failed，事务要不就在执行中，要不然就是成功或者失败的状态。

![3_state_of_transaction](../../resource/Database/RDBMS/3_state_of_transaction.png)

进一步放大看，事物内部还有部分提交这个中间状态，其对外是不可见的。

![5_state_of_transaction](../../resource/Database/RDBMS/5_state_of_transaction.png)

所以，具体来说，事务有以下几种可能的状态：

- Active：事务的初始状态，表示事务正在执行；
- Partially Committed：在最后一条语句执行之后；
- Failed：发现事务无法正常执行之后；
- Aborted：事务被回滚并且数据库恢复到了事务进行之前的状态之后；
- Committed：成功执行整个事务。

我们也可以看到，事务在执行之后只会以Aborted或者Committed状态作为结束。

# 2. ACID 特性

## 2.1 ACID 简介

为了保持数据库的一致性，在事务处理之前和之后，都遵循某些属性，也就是大家耳熟能详的ACID属性：

- 原子性（Atomicity）：即不可分割性，事务中的操作要么全不做，要么全做
- 一致性（Consistency）：一个事务在执行前后，数据库都必须处于正确的状态，满足[完整性约束](https://baike.baidu.com/item/数据完整性约束)
- 隔离性（Isolation）：多个事务并发执行时，一个事务的执行不应影响其他事务的执行
- 持久性（Durability）：事务处理完成后，对数据的修改就是永久的，即便系统故障也不会丢失

并非任意的对数据库的操作序列都是数据库事务。ACID属性是一系列操作组成事务的必要条件。总体而言，ACID属性提供了一种机制，使每个事务都”作为一个单元，完成一组操作，产生一致结果，事务彼此隔离，更新永久生效“，从而来确保数据库的正确性和一致性。

## 2.2 原子性（Atomicity）

原子性也被称为“全有或全无规则”。它非常好理解，即整个事务要么完整发生，要么根本不发生，不会部分发生。它涉及以下两个操作：

- **中止**：如果事务中止，则看不到对数据库所做的更改。
- **提交**：如果事务提交，则所做的更改可见。

拿之前转账的例子来说，用户A给用户B转账，至少要包含两个操作，用户A钱数减少，用户B钱数增加，增加和减少的操作要么全部成功，要么全部失败，是一个原子操作。如下图，如果事务在**T1** 完成之后但在**T2**完成之前失败，将导致数据库状态不正确。

![transaction_transfer](../../resource/Database/RDBMS/transaction_transfer.jpeg)

## 2.3 一致性（Consistency）

一致性是指，一个事务必须使数据库从一个一致性状态变换到另一个一致性状态（执行成功），或回滚到原始的一致性状态（执行失败）。这意味着必须维护完整性约束，以使在事务之前和之后数据库保持一致性和正确性。

参考上面的示例，假设用户A和用户B两者的钱加起来一共是700，那么不管A和B之间如何转账，转几次账，这一约束都得成立，即事务结束后两个用户的钱相加起来还得是700，这就是事务的一致性。

如果转账过程中，仅完成A扣款或B增款两个操作中的一个，即未保证原子性，那么结果数据如上述完整性约束也就无法得到维护，一致性也就被打破。可以看出，事务的一致性和原子性是密切相关的，原子性的破坏可能导致数据库的不一致。

但数据的一致性问题并不都和原子性有关。比如转账的过程中，用户A扣款了100，而用户B只收款了50，那么该过程可以符合原子性，但是数据的一致性就出现了问题。

**一致性既是事务的属性，也是事务的目的**。也正如本文开篇所提到的，“事务是用来确保无论发生什么情况，你使用的数据都将处于一个合理的状态“，这里所说的合理/正确，也就是指满足完整性约束。

总的来说，**一致性是事务ACID四大特性中最重要的属性，而原子性、隔离性和持久性，都是作为保障一致性的手段。事务作为这些性质的载体，实现了这种由ACID保障C的机制。**

![relation_of_acid](../../resource/Database/RDBMS/relation_of_acid.png)

### ACID和CAP中C（一致性）的区别

注意，我们一直在讨论的一致性，即ACID中的C，是指单一实体内部的正确状态在时间维度上的一致性，进一步说，是通过维护数据的完整性约束，来保持数据库在时间上（比如事务前后）保持一致的正确状态。因为是描述单一实体的内部状态，故又称“内部一致性”。

而CAP原则中的一致性是指在分布式系统中，空间维度上，某一特定时刻，多个实体中不同数据备份之间值的一致性，又称“外部一致性”。

## 2.4 隔离性（Isolation）

隔离性是指，并发执行的各个事务之间不能互相干扰，即一个事务内部的操作及使用的数据，对并发的其他事务是隔离的。**此属性确保并发执行一系列事务的效果等同于以某种顺序串行地执行它们**，也就是要达到这么一种效果：对于任意两个并发的事务T1和T2，在事务T1看来，T2要么在T1开始之前就已经结束，要么在T1结束之后才开始，这样每个事务都感觉不到有其他事务在并发地执行。这要求两件事:

- 在一个事务执行过程中，数据的中间的（可能不一致）状态不应该被暴露给所有的其他事务。
- 两个并发的事务应该不能操作同一项数据。数据库管理系统通常使用锁来实现这个特征。

还是拿转账来说，在A向B转账的整个过程中，只要事务还没有提交（commit），查询A账户和B账户的时候，两个账户里面的钱的数量都不会有变化。如果在A给B转账的同时，有另外一个事务执行了C给B转账的操作，那么当两个事务都结束的时候，B账户里面的钱必定是A转给B的钱加上C转给B的钱再加上自己原有的钱。

如此，隔离性防止了多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括未提交读（Read uncommitted）、提交读（read committed）、可重复读（repeatable read）和串行化（Serializable）。以上4个级别的隔离性依次增强，分别解决不同的问题。**事务隔离级别越高，就越能保证数据的完整性和一致性，但同时对并发性能的影响也越大**。

## 2.5 持久性（Durability）

事务的持久性又称为永久性（Permanency），是指一个事务一旦提交，对数据库中对应数据的状态变更就应该是**永久性**的。即使发生系统崩溃或机器宕机等故障，只要数据库能够重新启动，那么一定能够根据事务日志对未持久化的数据重新进行操作，将其**恢复**到事务成功结束的状态。持久性意味着在事务完成以后，该事务所对数据库所作的更改便持久的保存在数据库之中，并不会因为系统故障而被回滚。（完成的事务是系统永久的部分，对系统的影响是永久性的）

许多数据库通过引入**预写式日志**（Write-ahead logging，缩写 WAL）机制，来保证事务持久性和数据完整性，同时又很大程度上避免了基于事务直接刷新数据的频繁IO对性能的影响。

> 在使用WAL的系统中，所有的修改都先被写入到日志中，然后再被应用到系统状态中。假设一个程序在执行某些操作的过程中机器掉电了。在重新启动时，程序可能需要知道当时执行的操作是成功了还是部分成功或者是失败了。如果使用了WAL，程序就可以检查log文件，并对突然掉电时计划执行的操作内容跟实际上执行的操作内容进行比较。在这个比较的基础上，程序就可以决定是撤销已做的操作还是继续完成已做的操作，或者是保持原样。

# 3. 总结

事务（Transaction）是由一系列对系统中数据进行访问或更新的操作所组成的一个程序执行逻辑单元（Unit）。在事务的ACID特性中，C即一致性是事务的根本追求，而对数据一致性的破坏主要来自两个方面：

- 事务的并发执行
- 事务故障或系统故障

数据库系统是通过并发控制技术和日志恢复技术来避免这种情况发生的。

并发控制技术保证了事务的隔离性，使数据库的一致性状态不会因为并发执行的操作被破坏。

日志恢复技术保证了事务的原子性，使一致性状态不会因事务或系统故障被破坏。同时使已提交的对数据库的修改不会因系统崩溃而丢失，保证了事务的持久性。

![concurrency_control_log_recovery](../../resource/Database/RDBMS/concurrency_control_log_recovery.png)

## 参考文献

1. *What is a database transaction? (2019). Retrieved November 5, 2019, from Stack Overflow website: https://stackoverflow.com/questions/974596/what-is-a-database-transaction*
2. *Communcations and Information Processing: First International Conference, ICCIP 2012, Aveiro, Portugal, March 7-11, 2012, Proceedings, 第 2 部分*
3. *Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems, 第25节*
4. *ACID Properties in DBMS - GeeksforGeeks. (2016, August 7). Retrieved November 5, 2019, from GeeksforGeeks website: https://www.geeksforgeeks.org/acid-properties-in-dbms/*
5. *维基百科. (2011, July 25). 预写式日志. Retrieved November 5, 2019, from Wikipedia.org website: https://zh.wikipedia.org/wiki/预写式日志*
6. *数据库事务的概念及其实现原理 - takumiCX - 博客园. (2018). Retrieved November 5, 2019, from Cnblogs.com website: https://www.cnblogs.com/takumicx/p/9998844.html*
7. *浅入深出MySQL中事务的实现. (2017, August 20). Retrieved November 5, 2019, from 面向信仰编程 website: https://draveness.me/mysql-transaction*

