# Insert

## 0、MySQL架构

![214741-20200602082219657-1021694047](./assets/214741-20200602082219657-1021694047.png)

## 1、调用堆栈细节

```
#0  lock_rec_insert_check_and_lock (flags=0, rec=0x7f5d74504242 "\200", block=0x7f5d739ef528, index=0x7f5d0c022410,
    thr=0x7f5d0c00aba8, mtr=0x7f5d802099b0, inherit=0x7f5d80209278)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/lock/lock0lock.cc:5908
#1  0x0000000001b35b17 in btr_cur_ins_lock_and_undo (flags=0, cursor=0x7f5d80209ed0, entry=0x7f5d0c9410e0,
    thr=0x7f5d0c00aba8, mtr=0x7f5d802099b0, inherit=0x7f5d80209278)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/btr/btr0cur.cc:2969
#2  0x0000000001b36419 in btr_cur_optimistic_insert (flags=0, cursor=0x7f5d80209ed0, offsets=0x7f5d80209678,
    heap=0x7f5d802099a8, entry=0x7f5d0c9410e0, rec=0x7f5d80209648, big_rec=0x7f5d80209ec8, n_ext=0,
    thr=0x7f5d0c00aba8, mtr=0x7f5d802099b0)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/btr/btr0cur.cc:3209
#3  0x0000000001a0af9c in row_ins_clust_index_entry_low (flags=0, mode=2, index=0x7f5d0c022410, n_uniq=1,
    entry=0x7f5d0c9410e0, n_ext=0, thr=0x7f5d0c00aba8, dup_chk_only=false)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/row/row0ins.cc:2639
#4  0x0000000001a0ce47 in row_ins_clust_index_entry (index=0x7f5d0c022410, entry=0x7f5d0c9410e0, thr=0x7f5d0c00aba8,
    n_ext=0, dup_chk_only=false)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/row/row0ins.cc:3326
#5  0x0000000001a0d336 in row_ins_index_entry (index=0x7f5d0c022410, entry=0x7f5d0c9410e0, thr=0x7f5d0c00aba8)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/row/row0ins.cc:3464
#6  0x0000000001a0d890 in row_ins_index_entry_step (node=0x7f5d0c00a850, thr=0x7f5d0c00aba8)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/row/row0ins.cc:3614
#7  0x0000000001a0dbff in row_ins (node=0x7f5d0c00a850, thr=0x7f5d0c00aba8)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/row/row0ins.cc:3752
#8  0x0000000001a0e05c in row_ins_step (thr=0x7f5d0c00aba8)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/row/row0ins.cc:3888
#9  0x0000000001a2bb2d in row_insert_for_mysql_using_ins_graph (mysql_rec=0x7f5d0c00e220 "\020",
    prebuilt=0x7f5d0c009fc0)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/row/row0mysql.cc:1746
#10 0x0000000001a2c094 in row_insert_for_mysql (mysql_rec=0x7f5d0c00e220 "\020", prebuilt=0x7f5d0c009fc0)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/row/row0mysql.cc:1866
#11 0x00000000018db13a in ha_innobase::write_row (this=0x7f5d0c0170c0, record=0x7f5d0c00e220 "\020")
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/innobase/handler/ha_innodb.cc:7667
#12 0x0000000000f5c889 in handler::ha_write_row (this=0x7f5d0c0170c0, buf=0x7f5d0c00e220 "\020")
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/sql/handler.cc:8173
#13 0x0000000001772501 in write_record (thd=0x7f5d0c013040, table=0x7f5d0c9406b0, info=0x7f5d8020b210,
    update=0x7f5d8020b190) at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/sql/sql_insert.cc:1895
#14 0x000000000176f6de in Sql_cmd_insert::mysql_insert (this=0x7f5d0c93f800, thd=0x7f5d0c013040,
    table_list=0x7f5d0c93f028) at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/sql/sql_insert.cc:776
#15 0x0000000001775fe1 in Sql_cmd_insert::execute (this=0x7f5d0c93f800, thd=0x7f5d0c013040)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/sql/sql_insert.cc:3142
#16 0x00000000015535db in mysql_execute_command (thd=0x7f5d0c013040, first_level=true)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/sql/sql_parse.cc:3607
#17 0x0000000001558fcc in mysql_parse (thd=0x7f5d0c013040, parser_state=0x7f5d8020c560)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/sql/sql_parse.cc:5584
#18 0x000000000154e737 in dispatch_command (thd=0x7f5d0c013040, com_data=0x7f5d8020cd00, command=COM_QUERY)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/sql/sql_parse.cc:1492
#19 0x000000000154d64d in do_command (thd=0x7f5d0c013040)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/sql/sql_parse.cc:1031
#20 0x000000000167e2a5 in handle_connection (arg=0x3b68ef0)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/sql/conn_handler/connection_handler_per_thread.cc:313
#21 0x0000000001d3b8d4 in pfs_spawn_thread (arg=0x3bad5c0)
    at /mnt/newdata/studio/mysql/mysql7/code/mysql-5.7.41/storage/perfschema/pfs.cc:2197
#22 0x00007f5d8ce02be0 in start_thread (arg=<optimized out>) at pthread_create.c:486
#23 0x00007f5d8b5803bf in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```

## 2、函数调用过程

![Insert插入流程](./assets/Insert%E6%8F%92%E5%85%A5%E6%B5%81%E7%A8%8B.jpg)

## 3、函数调用细节

### 1、do_command

```c++
// sql/sql_parse.cc:901
/**
  Read one command from connection and execute it (query or simple command).
  This function is called in loop from thread function.

  For profiling to work, it must never be called recursively.

  @retval
    0  success
  @retval
    1  request of thread shutdown (see dispatch_command() description)
*/
/**
  从连接中读取一条命令并执行（查询或简单命令）。
  该函数由线程函数循环调用。

  为使剖析有效，绝不能递归调用该函数。
  
  @retval
    0 成功
  @retval
    1 请求关闭线程（参见 dispatch_command() 说明）
*/
bool do_command(THD *thd){
  ...
  /*
    indicator of uninitialized lex => normal flow of errors handling
    (see my_message_sql)
  */
  /*
  	未初始化 lex 的指示器 => 错误处理的正常流程
    (见 my_message_sql）
  */
  thd->lex->set_current_select(0);

}
```

 

