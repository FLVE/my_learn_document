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





