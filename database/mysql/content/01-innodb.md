# InnoDB存储引擎

## 1、体系架构



### 1、后台线程

> InnoDB是多线程的模型，每个线程负责不同的任务。

#### 1、Master Thread

核心线程

负责：缓冲池中数据异步刷新到磁盘，保障数据的一致性等。



#### 2、IO Thread

使用AIO处理写IO请求，可以提高数据库的性能。

InnoDB1.0之前有4个IO Thread：write、read、insert buffer、log IO thread，并且使用innodb_file_io_threads增大IO Thread;之后write和read分别扩到了4个，并且使用innodb_read_io_threads和innodb_write_io_threads参数进行调整。

```sql
-- 查看innodb版本
MySQL [information_schema]> show variables like 'innodb_version';
+----------------+--------+
| Variable_name  | Value  |
+----------------+--------+
| innodb_version | 5.7.18 |
+----------------+--------+

-- 查看innodb线程数
MySQL [information_schema]> show variables like 'innodb_%io_threads';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| innodb_read_io_threads  | 12    |
| innodb_write_io_threads | 12    |
+-------------------------+-------+

--查看IO Threads
MySQL [information_schema]> show engine innodb status\G
*************************** 1. row ***************************
  Type: InnoDB
  Name: 
Status: 
=====================================
2023-09-18 19:45:04 0x7ef76d19f700 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 4 seconds
-----------------
BACKGROUND THREAD
-----------------
srv_master_thread loops: 26218631 srv_active, 0 srv_shutdown, 19545351 srv_idle
srv_master_thread log flush and writes: 45762617
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 547932430
OS WAIT ARRAY INFO: signal count 1588692579
RW-shared spins 0, rounds 2761478251, OS waits 266291494
RW-excl spins 0, rounds 32431875051, OS waits 179399983
RW-sx spins 1704818902, rounds 5631521605, OS waits 20224692
Spin rounds per wait: 2761478251.00 RW-shared, 32431875051.00 RW-excl, 3.30 RW-sx
------------
TRANSACTIONS
------------
Trx id counter 11564561404
Purge done for trx's n:o < 11564561404 undo n:o < 0 state: running but idle
History list length 37
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 421111133548792, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133580288, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133579272, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133578256, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133577240, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133576224, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133575208, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133574192, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133573176, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133572160, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133571144, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133570128, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133569112, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133568096, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133567080, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133566064, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133565048, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133564032, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133563016, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133562000, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133560984, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133559968, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133558952, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133557936, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133556920, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133555904, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133554888, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133553872, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133551840, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133550824, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133549808, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133547776, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133546760, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133545744, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133544728, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133543712, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133542696, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133541680, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133540664, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133539648, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133538632, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133537616, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133536600, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133534568, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133535584, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133533552, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133531520, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133532536, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133530504, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133529488, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133527456, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133528472, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133525424, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133524408, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133521360, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133523392, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133522376, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133520344, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133519328, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133516280, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133518312, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133517296, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133502056, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133515264, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133514248, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133513232, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133512216, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133511200, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133510184, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133509168, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133508152, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133507136, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133505104, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133504088, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133503072, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133501040, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133500024, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133499008, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133497992, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133496976, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133495960, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133494944, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133493928, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133492912, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133491896, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133490880, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133489864, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133487832, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133486816, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133485800, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133484784, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133483768, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133482752, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133481736, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133480720, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133479704, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133478688, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133477672, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133476656, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133475640, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133474624, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133473608, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133472592, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133471576, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133470560, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133469544, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133468528, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133467512, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133466496, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133465480, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133464464, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133463448, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133462432, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133461416, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133460400, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133459384, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133458368, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133457352, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133456336, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133455320, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133454304, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133453288, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133452272, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133451256, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133448208, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133450240, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421111133449224, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 11564561401, ACTIVE 0 sec
2 lock struct(s), heap size 1136, 1 row lock(s), undo log entries 1
MySQL thread id 8982349, OS thread handle 139601468077824, query id 15787028683 10.0.12.16 db_jf
---TRANSACTION 11564561398, ACTIVE 0 sec
2 lock struct(s), heap size 1136, 1 row lock(s), undo log entries 1
MySQL thread id 9032533, OS thread handle 139612365776640, query id 15787028680 10.0.12.16 db_jf
---TRANSACTION 11564561395, ACTIVE 0 sec
2 lock struct(s), heap size 1136, 1 row lock(s), undo log entries 1
MySQL thread id 8982325, OS thread handle 139612523063040, query id 15787028677 10.0.12.16 db_jf
---TRANSACTION 11564561392, ACTIVE 0 sec preparing
2 lock struct(s), heap size 1136, 1 row lock(s), undo log entries 1
MySQL thread id 9032586, OS thread handle 139620358551296, query id 15787028687 10.0.12.16 db_jf starting
COMMIT
--------
FILE I/O
--------
I/O thread 0 state: waiting for completed aio requests (insert buffer thread)
I/O thread 1 state: waiting for completed aio requests (log thread)
I/O thread 2 state: waiting for completed aio requests (read thread)
I/O thread 3 state: waiting for completed aio requests (read thread)
I/O thread 4 state: waiting for completed aio requests (read thread)
I/O thread 5 state: waiting for completed aio requests (read thread)
I/O thread 6 state: waiting for completed aio requests (read thread)
I/O thread 7 state: waiting for completed aio requests (read thread)
I/O thread 8 state: waiting for completed aio requests (read thread)
I/O thread 9 state: waiting for completed aio requests (read thread)
I/O thread 10 state: waiting for completed aio requests (read thread)
I/O thread 11 state: waiting for completed aio requests (read thread)
I/O thread 12 state: waiting for completed aio requests (read thread)
I/O thread 13 state: waiting for completed aio requests (read thread)
I/O thread 14 state: waiting for completed aio requests (write thread)
I/O thread 15 state: waiting for completed aio requests (write thread)
I/O thread 16 state: waiting for completed aio requests (write thread)
I/O thread 17 state: waiting for completed aio requests (write thread)
I/O thread 18 state: waiting for completed aio requests (write thread)
I/O thread 19 state: waiting for completed aio requests (write thread)
I/O thread 20 state: waiting for completed aio requests (write thread)
I/O thread 21 state: waiting for completed aio requests (write thread)
I/O thread 22 state: waiting for completed aio requests (write thread)
I/O thread 23 state: waiting for completed aio requests (write thread)
I/O thread 24 state: waiting for completed aio requests (write thread)
I/O thread 25 state: waiting for completed aio requests (write thread)
Pending normal aio reads: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0] , aio writes: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0] ,
 ibuf aio reads:, log i/o's:, sync i/o's:
Pending flushes (fsync) log: 0; buffer pool: 0
12922746591 OS file reads, 8763434376 OS file writes, 550263684 OS fsyncs
28.74 reads/s, 16384 avg bytes/read, 503.12 writes/s, 44.24 fsyncs/s
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 20198, seg size 20200, 517198472 merges
merged operations:
 insert 970657549, delete mark 194097125, delete 7208020
discarded operations:
 insert 0, delete mark 1, delete 0
Hash table size 1593833, node heap has 0 buffer(s)
Hash table size 1593833, node heap has 0 buffer(s)
Hash table size 1593833, node heap has 0 buffer(s)
Hash table size 1593833, node heap has 0 buffer(s)
Hash table size 1593833, node heap has 0 buffer(s)
Hash table size 1593833, node heap has 0 buffer(s)
Hash table size 1593833, node heap has 0 buffer(s)
Hash table size 1593833, node heap has 0 buffer(s)
0.00 hash searches/s, 31644.09 non-hash searches/s
---
LOG
---
Log sequence number 26723180487931
Log flushed up to   26723180455745
Pages flushed up to 26722807772451
Last checkpoint at  26722793250564
0 pending log flushes, 0 pending chkp writes
4533038065 log i/o's done, 3.25 log i/o's/second
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 6055526400
Dictionary memory allocated 4857920
Buffer pool size   360448
Free buffers       4019
Database pages     356429
Old database pages 131532
Modified db pages  45224
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 5303914646, not young 810909501652
28.74 youngs/s, 102.47 non-youngs/s
Pages read 12925030806, created 1440598537, written 4039927404
28.74 reads/s, 119.72 creates/s, 482.88 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 16.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 356429, unzip_LRU len: 0
I/O sum[47796]:cur[3300], unzip sum[0]:cur[0]
----------------------
INDIVIDUAL BUFFER POOL INFO
----------------------
---BUFFER POOL 0
Buffer pool size   90112
Free buffers       1021
Database pages     89091
Old database pages 32867
Modified db pages  12325
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 1324532177, not young 202596075793
7.25 youngs/s, 28.49 non-youngs/s
Pages read 3268855222, created 362710407, written 1130595385
4.75 reads/s, 29.74 creates/s, 120.72 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 89091, unzip_LRU len: 0
I/O sum[11949]:cur[825], unzip sum[0]:cur[0]
---BUFFER POOL 1
Buffer pool size   90112
Free buffers       960
Database pages     89152
Old database pages 32928
Modified db pages  10459
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 1337659927, not young 204347325369
9.00 youngs/s, 29.99 non-youngs/s
Pages read 3214187404, created 359217454, written 981280348
19.50 reads/s, 7.00 creates/s, 120.72 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 1 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 89152, unzip_LRU len: 0
I/O sum[11949]:cur[825], unzip sum[0]:cur[0]
---BUFFER POOL 2
Buffer pool size   90112
Free buffers       1017
Database pages     89095
Old database pages 32868
Modified db pages  11244
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 1343108886, not young 201336715796
6.25 youngs/s, 24.99 non-youngs/s
Pages read 3218564490, created 359476315, written 956579127
1.75 reads/s, 42.74 creates/s, 120.72 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 89095, unzip_LRU len: 0
I/O sum[11949]:cur[825], unzip sum[0]:cur[0]
---BUFFER POOL 3
Buffer pool size   90112
Free buffers       1021
Database pages     89091
Old database pages 32869
Modified db pages  11196
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 1298613656, not young 202629384694
6.25 youngs/s, 19.00 non-youngs/s
Pages read 3223423690, created 359194361, written 971472544
2.75 reads/s, 40.24 creates/s, 120.72 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 16.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 89091, unzip_LRU len: 0
I/O sum[11949]:cur[825], unzip sum[0]:cur[0]
--------------
ROW OPERATIONS
--------------
0 queries inside InnoDB, 0 queries in queue
0 read views open inside InnoDB
Process ID=346881, Main thread ID=139628642191104, state: sleeping
Number of rows inserted 59140912719, updated 4119839601, deleted 15352723, read 922880430201
5595.35 inserts/s, 928.77 updates/s, 0.00 deletes/s, 1073.48 reads/s
----------------------------
END OF INNODB MONITOR OUTPUT
============================
```

#### 3、Purge Thread

事务被提交后，使用的undo log不再需要，需要purge thread来回收已经使用并分配的undo页。innodb1.1后，purge操作可以独立到单独的线程进行，不再master thread中执行，以减轻master thread的压力。

通过以下参数设置可以启用独立的purge thread

```sql
[mysqld]
innodb_purge_threads=1
```





