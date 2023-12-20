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
......
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
......
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

InnoDB1.2以后，支持多个Purge Thread，进一步加快undo页的回收

```sql
MySQL [information_schema]> show variables like 'innodb_purge_threads';
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| innodb_purge_threads | 4     |
+----------------------+-------+
```

#### 4、Page Cleaner Thread

将脏页的刷新操作都放入单独的线程中完成，减轻Master Thread的工作及对于用户查询线程的阻塞。



### 2、内存

#### 1、缓冲池

```sql
MySQL [information_schema]> show variables like 'innodb_buffer_pool_size';
+-------------------------+------------+
| Variable_name           | Value      |
+-------------------------+------------+
| innodb_buffer_pool_size | 5905580032 |
+-------------------------+------------+
```



```sql
MySQL [information_schema]> show variables like 'innodb_buffer_pool_instances';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| innodb_buffer_pool_instances | 4     |
+------------------------------+-------+

MySQL [information_schema]> show engine innodb status\G
*************************** 1. row ***************************
  Type: InnoDB
......
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 6055526400
Dictionary memory allocated 4862653
Buffer pool size   360448
Free buffers       4096
Database pages     356352
Old database pages 131464
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 5304317505, not young 810930277821
0.00 youngs/s, 0.00 non-youngs/s
Pages read 12925517945, created 1446008302, written 4063805353
0.00 reads/s, 0.00 creates/s, 1.50 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 356352, unzip_LRU len: 0
I/O sum[5020]:cur[0], unzip sum[0]:cur[0]
----------------------
INDIVIDUAL BUFFER POOL INFO
----------------------
---BUFFER POOL 0
Buffer pool size   90112
Free buffers       1024
Database pages     89088
Old database pages 32866
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 1324647530, not young 202603934065
0.00 youngs/s, 0.00 non-youngs/s
Pages read 3268981677, created 364059434, written 1139501556
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 89088, unzip_LRU len: 0
I/O sum[1255]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 1
Buffer pool size   90112
Free buffers       1024
Database pages     89088
Old database pages 32866
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 1337756599, not young 204351418390
0.00 youngs/s, 0.00 non-youngs/s
Pages read 3214307952, created 360571896, written 986781302
0.00 reads/s, 0.00 creates/s, 1.50 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 89088, unzip_LRU len: 0
I/O sum[1255]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 2
Buffer pool size   90112
Free buffers       1024
Database pages     89088
Old database pages 32866
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 1343202648, not young 201342091839
0.00 youngs/s, 0.00 non-youngs/s
Pages read 3218682305, created 360828263, written 961119840
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 89088, unzip_LRU len: 0
I/O sum[1255]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 3
Buffer pool size   90112
Free buffers       1024
Database pages     89088
Old database pages 32866
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 1298710728, not young 202632833527
0.00 youngs/s, 0.00 non-youngs/s
Pages read 3223546011, created 360548709, written 976402655
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 89088, unzip_LRU len: 0
I/O sum[1255]:cur[0], unzip sum[0]:cur[0]

.......................
END OF INNODB MONITOR OUTPUT
============================


-- sql命令查询
MySQL [information_schema]> select pool_id,pool_size,free_buffers,database_pages from information_schema.innodb_buffer_pool_stats\G
*************************** 1. row ***************************
       pool_id: 0
     pool_size: 90112
  free_buffers: 1024
database_pages: 89088
*************************** 2. row ***************************
       pool_id: 1
     pool_size: 90112
  free_buffers: 1024
database_pages: 89088
*************************** 3. row ***************************
       pool_id: 2
     pool_size: 90112
  free_buffers: 1024
database_pages: 89088
*************************** 4. row ***************************
       pool_id: 3
     pool_size: 90112
  free_buffers: 1024
database_pages: 89088
4 rows in set (0.001 sec)
```



## 2、Checkpoint

Write Ahead Log策略：当事务提交时，先写重做日志，再修改页。

Chedckpoint目的：

> 1、缩短数据库的恢复时间
>
> 2、缓冲池不够用时，将脏页刷新到磁盘
>
> 3、重做日志不可用时，刷新脏页

宕机时，只用恢复Checkpoint后的重做日志，缩短了恢复时间。

InnoDB时通过LSN标记版本。每个页有LSN，重做日志也有LSN，Checkpoint也有LSN

```sql
MySQL [information_schema]> show engine innodb status\G
*************************** 1. row ***************************
  Type: InnoDB
  Name: 
Status: 
=====================================
2023-09-27 15:07:22 0x7ef77dde2700 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 2 seconds
.........
---
LOG
---
Log sequence number 26841334239593
Log flushed up to   26841334239593
Pages flushed up to 26841334239593
Last checkpoint at  26841334239584
0 pending log flushes, 0 pending chkp writes
4533819078 log i/o's done, 2.00 log i/o's/second
.....
----------------------------
END OF INNODB MONITOR OUTPUT
============================
```



两种Checkpoint：

1、Sharp Checkpoint：数据库关闭时，将所有脏页刷新，默认方式，即参数：innodb_fast_shutdown=1

2、FUzzy Checkpoint：刷新一部分脏页



InnoDB 可能发生几种情况的Fuzzy Checkpoint

1、Mater Thread Checkpoint

​		以每秒/每十秒从缓冲池的脏页列表中刷新一定比例的页回磁盘，异步操作，不会阻塞用户查询

2、FLUSH_LRU_LIST Checkpoint

​		InnoDB需要保证LRU列表中需要超不多100个空闲页可供使用，会阻塞用户查询操作。如果可用操作的空闲页，从LRU列表末端页一尺，存在脏页就会需要Checkpoint

​		从5.6版本后，检查空闲页被单独放在了Page CLeaner线程中，可以通过参数innodb_lru_scan_depth控制LRU列表中可用页的数量，默认为1024

```sql
MySQL [information_schema]> show variables like 'innodb_lru_scan_depth';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_lru_scan_depth | 1024  |
+-----------------------+-------+
```

3、Async/sync Flush Checkpoint

​		重做日志不可用，需强制将一些页刷新回磁盘。如将已经写入到重做日志的LSN记录为redo_lsn,将已经刷新回磁盘最新页的LSN几位Checkpoint_lsn，存在以下定义

​		**checkpoint = redo_lsn - checkpoint_lsn**

​		async_water_mark = 75% * total_redo_log_file_size

​		sync_water_mark = 90% * total_redo_log_file_size

​		假设设置两个重做日志文件，每个为1G，async_water_mark为1.5G,sync_water_mark为1.8G

​		当checkpoint_age < async_wter_mark,不需要刷新任何脏页到磁盘

​		当async_wter_mark < checkpoint_age < sync_water_makr，触发async Flush。

​		5.6版本后该操作从Mater Thread线程中独立到Page Cleaner Thread中，不阻塞用户操作

4、Dirty Page too much

​		脏页数据太多，保证缓冲池中有足够可用的页，有参数innodb_max_dirty_pages_pct控制，默认为75%

```sql
MySQL [information_schema]> show variables like 'innodb_max_dirty_pages_pct';
+----------------------------+-----------+
| Variable_name              | Value     |
+----------------------------+-----------+
| innodb_max_dirty_pages_pct | 75.000000 |
+----------------------------+-----------+
```

## 3、Master Thread 工作方式

### 1、InnoDB1.0.X前

Master Thread 具有最高的线程优先级别，其内部有多个循环组成：主循环、后台循环、刷新循环、暂停循环

```c
void master thread() {
	goto loop;
1oop: // 主循环
for(int i = 0; i<10; 1++){
    thread sleep(1) // sleep 1 second
    do log buffer flush to disk // 日志缓冲刷新到磁盘
    if ( last one second ios < 5 )
    	do merge at most 5 insert buffer //合并插入缓存
    if ( buf get modifled ratio pct > innodb max dirty pages pct
    	do buffer pool fush 100 dirty page // 刷新100个脏页
    if ( no user activity )
    	goto backgroud loop // 跳转到后台循环
if ( last ten second ios < 200 )
	do buffer pool flush 100 dirty page //刷100个脏页
do merge at most 5 insert buffer //合并插入缓存
do log buffer flush to disk //日志缓冲刷新到磁盘
do full purge //删除无用Undo页
if ( buf get_modified ratio_pct > 70% )
	do buffer pool flush 100 dirty page //刷新100个脏页
else
	buffer pool flush 10 dirty page //刷新10个脏页
goto loop
background loop:// 后台循环
do full purge // 删除无用Undo页
do merge 20 insert buffer //合并插入缓存
if not idle:
goto loop:
else:
	goto flush loop
flush loop:// 刷新循环
do buffer pool flush 100 dirty page //刷新100个脏页
if ( buf get modified ratio_pct>innodb _max_dirty_pages_pct )
	goto flush loop
goto suspend loop
suspend loop:# 暂停循环
suspend thread()
waiting event
goto loop;
}
```

### 2、InnoDB1.2.x之前

刷新脏页(100)和合并插入缓存(20)太少，当宕机时恢复时间可能更长。

为解决该问题，新增了innodb_io_capacity参数，默认200.具有如下规则：

1、合并插入缓存时，数量为该参数设置的5%。

2、刷新脏页时，数量为该参数设置的值。

 

原规则：脏页所占比例 < innodb_max_diry_pages_pct,不刷新。大于时，刷新100个脏页。

使用新参数的规则：buf_flush_get_desird_flush_rate函数通过判断重做日志的速度决定最合适的刷新脏页数量，可能脏页比例小于设置的值，也会刷新一定量的脏页。

之前full purge最多回收20个undo页，1.0.x后引入了参数innodb_purge_batch_size可用控制每次full purge回收的Undo页，默认为20。

```sql
MySQL [information_schema]> show variables like 'innodb_purge_batch_size';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| innodb_purge_batch_size | 300   |
+-------------------------+-------+

# 设置回收的Undo页数量
set global innodb_purge_batch_size=x;
```

```c
void master thread(){
goto loop;
loop:
for(int i m 0; i<10; i++)[
    thread sleep(1) // sleep 1 second
    do loq buffer flush to disk
    if ( last_one_second_ios < 5% innodb_io_capacity )
    	do merge 5% innodb_io_capacity insert buffer
    if ( buf_get_modified_ratio_pct > innodb_max_dirty_pages_pct )
    	do buffer pool flush 100% innodb_io_capacity dirty page
    else if enable adaptive flush
    	do buffer pool flush desired amount dirty page
    if ( no user activity )
    	goto backgroud loop
if ( last_ten_second_ios <innodb_io_capacity)
	do buffer pool flush 100% innodb_io_capacity dirty page
do merge 5% innodb_io_capacity insert buffer
do log buffer flush to disk
do full purge
if ( buf_get_modified_ratio_pct > 70% )
	do buffer pool flush 100% innodb_io_capacity dirty page
else
	do buffer pool flush 10% innodb_io_capacity dirty page
goto 1oop
background loop:
do full purge
do merge 100% innodb_io_capacity insert buffer
if not idle:
goto loop:
else:
	goto flush loop
flush loop:
do buffer pool flush 100% innodb_io_capacity dirty page
if ( buf get modified ratio pct>innodb max dirty pages pct )
	go to flush 1oop
	goto suspend loop
suspend loop:
suspend thread()
waiting event
goto loop;
}
```

3、1.2.x版本

```c
if Innodb is idle
    src_master_do_idle_tasks();//每10秒的操作
else
    src_master_do_active_tasks();// 每秒的操作
```

刷脏页的操作从Master Thread线程分离一个单独的page Cleaner Thread,减轻主线程的工作。

## 4、关键特性

### 1、插入缓冲

#### 1、insert Buffer

插入聚集索引一般是顺序的，不需要磁盘的随机读取
但插入非聚集索引叶子节点不是顺序的，需要离散访问非聚集索引页，速度较慢。
对于非聚集索引的插入或更新，先判断插入的非聚集索引页是否在缓存池中，若在，直接插入，或不在，先放到一个Inser Buffer对象中，
然后根据一些算法将Insert Buffer缓存的记录通过后台线程慢慢合并刷新回辅助索引。
插入缓冲将多次插入合并为一次操作，减少磁盘的离散操作。

使用Insert Buffer需满足两个条件：
1、索引是辅助索引
2、索引不是唯一的（不需要查找索引页判断唯一性）

#### 2、change Buffer

1.0.x引入了change buffer，视为Insert buffer的加强版，适用对象为非唯一的辅助索引。

change buffer包括：insert buffer、delete buffer、purge buffer。

一条记录进行update操作可能分为两个过程：

1、将记录标记为删除

2、真正将记录删除

1.2.x后，可以通过参数innodb_change_buffer_max_size控制change buffer最大使用内存数量

```	sql
MySQL [information_schema]> show variables like 'innodb_change_buffer_max_size';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| innodb_change_buffer_max_size | 25    |
+-------------------------------+-------+
```

```sql
MySQL [information_schema]> show engine innodb status\G
------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 20198, seg size 20200, 517202584 merges
merged operations:
 insert 970680436, delete mark 194097755, delete 7208024
discarded operations:
 insert 0, delete mark 1, delete 0
```

insert表示insert buffer；delete mark表示delete buffer；delete表示purge buffer

discarded operator表示当change buffer发生merge时，表已经被删除，就无需记录合并到辅助索引中

### 2、两次写



### 3、自适应哈希索引



### 4、异步IO



### 5、刷新邻接页
