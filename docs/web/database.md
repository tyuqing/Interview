# 数据库相关

## 应用

### 十万条数据插入数据库，怎么去优化和处理高并发情况下的DB插入

首先是优化方面，对于十万条数据，如果处理每条数据时构建一个实例化对象，处理具体业务逻辑，然后进行一次io操作，十万条数据要构建十万个对象，操作十万次io，对于内存和io方面来说这显然是不现实的。整体的思路可以考虑**构建一个生产消费者模式，即一个生产队列，一个消费队列**。十万条数据可以到生产队列中依次排队，每条数据出队列时先到**实例享元池**中进行过滤（每条数据在处理完成后进行一次对象回收，回收到享元池中，下一条数据发现享元池中已经存在该对象就不重新构建而是拿来直接使用，可以**避免在两次gc之间瞬时内存过大的情况**）。假设这十万条数据可能分属十张表中，那么在每张表之前构建一个消费队列，从生产队列出来的数据到各自的消费队列中依次进行业务处理，处理完的数据归属到各个表的内存中（如果只考虑插入数据的话基本包括添加、更新、删除这几个操作，对于一些异常情况，例如后面的操作先进入了队列，可以将该数据放入队列尾部重新排队。

如果考虑高并发的情况，也就是在**插入数据时同时要读取数据，那么需要构建两个消费队列，即一个读取队列，一个插入队列**，两个队列为互斥的，对于这种情况处理就会更加复杂）。当每张表的数据处理完成后，将该表进行一次io操作，把内存中的数据同步更新到数据库中。**当然如果考虑一些容灾或者宕机的场景，可以将内存中的数据每隔一段时间进行一次redis备份**。

### 数据库中什么是 left join 和 right join 有什么区别（todo）



## 原理

### 数据库索引中为什么要用 Btree（todo）

### 什么是聚簇索引（todo）
