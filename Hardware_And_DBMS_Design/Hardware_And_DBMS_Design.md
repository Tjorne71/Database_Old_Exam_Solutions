# Hardware and DBMS Design

[All Answers](../All_Answers.md)

## August 2022

**a) Select the correct statements below:**

(a) Storing relations by columns can give significant performance boost for analytic queries. (50%)

(b) In a main-memory-based database system, some persistent storage is needed in case the server crashes. (50%)

(c) Documents in document stores always correspond to data from a single relation in a relational system. (−50%)

(d) Eventual consistency guarantees that all clients always get the latest version of data items. (−50%)

**b) Discuss pros and cons of key-value stores.**

- Key-value stores have a very simple data model and can thus be used to store any data. Since the key should be unique, they also lend themselves well to distributed applications. The drawback is that since they know nothing of the contents of the value, they have no query capacity.

## Maj 2022

**a) Select the correct statements below:**

(a) A B+-tree index could be used to index attribute values in a single-server document store. (50%)

(b) Consistent hashing is useful for workload balancing in a large distributed system. (50%)

(c) In a main-memory-based database system, there is no need for persistent storage of any kind. (−50%)

(d) The default isolation mode of PostgreSQL is full serializability. (−50%)

**b) In your own words, explain the original CAP theorem and why it is not very useful for system developers.**

- In a distributed system with multiple replicas of data items, if the network becomes partitioned, the system cannot both return consistent data (consistency) and return data immediately (availability). Since networks are mostly not partitioned during regular processing, and since the CAP theorem does not explain what to do in this case, the theorem has limited importance.

## December 2021

**a) Select the correct statements below:Ø**

(a) In a main-memory relational database system, it remains necessary to log the “old” values during updates, in case the transaction aborts. (50%)

(b) NoSQL literally means that SQL-like query languages must not be supported by NoSQL systems. (−50%)

(c) Document stores cannot store business data that is typically stored in relational systems. (−50%)

(d) While locking whole tables guarantees serializability in a relational system, it leads to terrible performance, which is why it is turned off by default in most systems. (50%)

**b) Discuss pros and cons of storing multiple copies of data in a geographically distributed database, compared to storing a single copy. Be sure to mention at least one advantage and one disadvantage.**

Pros: Reduces likelihood of losing data; Increases availability in case of network partitions. Cons: Increases the work required to maintain consistency, especially if strict consistency is desired; Increases storage requirements.

## August 2021

**a) Consider a database server with persistent main memory and no secondary storage. De ne the STEAL bu er management policy and discuss its usefulness for such a server.**

- The STEAL buffer management policy allows transactions to reuse buffer space with updated pages, by writing the pages to disk. Since the pages can simply be updated in the persistent memory, with this new server, STEAL simply does not apply.

**b) Select the true statements:**

(a) Writing the log is usually slower than writing the actual updates to relations. (0%)

(b) SSD storage is volatile. (0%)

(c) The FORCE policy is di cult to implement in reality, since committing large transactions can cause many disk writes, and the server might crash in the middle of the writing process. (50%)

(d) Query optimisation are often improved by gathering statistics over relations. (50%)

## June 2021

**6a) Argue why a buffer manager is needed in an HDD-based DBMS., Write your reflections here:**

- The crux of the answer should be: Data must be in memory before it is processed. Data is transferred in fixed size units between disk and RAM. Buffering avoids repeating expensive I/Os. Partial points 16 were awarded for discussing high cost of IOs, strategies for reducing IO cost, managing updates and maintaining transactional properties.

**6b) What is the query optimizer component responsible for? Select all statements that apply:**

(a) Generating a query execution plan. (50%)

(b) Parsing queries. (0%)

(c) Comparing access methods. (50%)

(d) Maintaining statistics over existing relations. (0%)

## March 2021

**6a) Select the true statements:**

(a) A lock manager is necessary to implement ACID-style isolation. (50%)

(b) When a transaction is committed, all the pages it has changed must be written to disk. (0%)

(c) A distributed system with multiple replicas of data items can provide both strict consistency and availability at the same time. (0%)

(d) A nested-loops join implementation is necessary to evaluate joins with complicated conditions. (50%)

**6b) Write your reections here:**

- In this case, we can consider the URL as a key, and the web-page as the value. Then the workload perfectly fits the strengths of a distributed key-value store. While other systems could possibly support the workload, they would have unneeded functionalities, which would be likely to reduce performance.

## Januar 2021

**6a) Select the true statements:**

(a) Database backups are needed in case secondary storage (HDDs or SSDs) fails. (50%)

(b) Many NoSQL systems offer distributed storage. (50%)

(c) CAP-style consistency is actually very similar to ACID-style consistency. (0%)

(d) An RDBMS which implements clustered indexes, can have multiple such clus- tered indexes per table. (0%)

**6b) Write your reflections here:**

- Example answer: Consider a scenario where a data value is changed in Denmark, and then shortly after read in China. There are two basic methods for ensuring consistency: (i) keeping one replica, in which case one or both of these requests must travel quite far, resulting in latency; or (ii) allowing many replicas, but ensuring that the operations are consistent. Consider a case where there are two replicas, one in China and one in Denmark. Then neither request has to travel far to find data, but making sure that the data read in China is correct, requires the systems to communicate between the replicas over the network, which produces latency.)

## August 2020

**6a) Select the true statements:**

(a) Main memory cannot break or fail, so database backups are not needed for main-memory database systems. (0%)

(b) CAP-style consistency is actually very similar to ACID-style isolation. (50%)

(c) Defragmentation (reorganizing files together on disk) does not make much sense on SSDs. (50%)

(d) By definition, a relational database can only use B+-tree indexes. (0%)

**6b) Write your reflections here:**

- Since there is lag on the network, it is quite possible that results are outdated by the time they reach the app. Hence, the app cannot be considered to have strongly consistent data.

## April 2020

**a) Select the correct statements below:**

(a) Storing multiple database recovery logs on a single hard disk drive (HDD) gives excellent logging performance. (0%)

(b) Data replication in a distributed system reduces the risk of losing data. (50%)

(c) The notion of consistency in the CAP theorem is the same as the notion of consistency in the ACID properties. (0%)

(d) Compared to hard-disk drive (HDD) technology, state-of-the-art solid state disks (SSDs) generally improve the performance of disk operations. (50%)

**b) Social media providers, such as Facebook, use multiple copies of data and eventual consistency for most of their functionality. As an example, comments on posts may not be seen immediately, or even in the same order, by all users. Discuss briefly how performance would be a ected if strong consistency were adopted instead. Base your discussion on a concrete example, such as the timing and order of comments described above.**

- There are two ways to move to strong consistency.
- a) A single copy is stored, and all updates happen to that copy. This would mean serious bottlenecks of access, as all accesses and updates to popular material would go to the same server.
- b) All copies are kept up to date. This would mean that updates and/or reads must access many copies simultaneously, which also would result in serious bottlenecks of access.

## Maj 2020

**a) Select the correct statements below:**

(a) Magnetic tapes are useful for archival storage (e.g., database backups). (100%)

(b) Utilisation of the L2 cache is not important in a disk-based database management system. (0%)

(c) Traditional relational systems can scale-up in nitely. (0%)

(d) By de nition, a NoSQL system can not implement the SQL query language. (0%)

**b) Discuss briefly the suitability of eventual consistency for a typical banking application.**

- Banking applications usually work with structured data and require ACID consistency, which is not only about individual values but relationships between values, including auditable counters and such. Eventual consistency is a much simpler concept,
- (a) focusing solely on individual records as discussed in the CAP theorem, and
- (b) even allowing inconsistencies in the values of those records. As such, Eventual Consistency is not at all suitable in banking.
