
## 对于SQL慢查询的优化
主要是从查询语句和数据库表设计两个方面来考虑，查询语句方面可以增加索引，增加查询筛选的限制条件；数据库表设计的时候可以拆分表，设计得更细粒度。但是后来才发现面试官想要的就是查询大量数据的慢查询问题的优化。

## mysql数据库存储引擎及其优缺点

## 优化数据库的方法，从sql到缓存到cpu到操作系统，知道多少说多少

数据库层级：
1. sql优化：尽量避免全表扫描如like、in；查询条件优化where、order by；避免selec*
2. 索引优化;对适当的字段添加索引
3. 表设计优化：范式、反范式；字段大小在符合应用场景的情况下尽量小
4. 数据库架构优化：读写分离；分库分表；添加redis缓存
5. 数据库配置优化：max_connections；sort_buffer_size；open_files_limit

系统内核：
net.ipv4.tcp_fin_timeout = 30  
#TIME_WAIT超时时间，默认是60s  
net.ipv4.tcp_tw_reuse = 1      
#1表示开启复用，允许TIME_WAIT socket重新用于新的TCP连接，0表示关闭  
net.ipv4.tcp_tw_recycle = 1    
#1表示开启TIME_WAIT socket快速回收，0表示关闭    
net.ipv4.tcp_max_tw_buckets = 4096     
#系统保持TIME_WAIT socket最大数量，如果超出这个数，系统将随机清除一些TIME_WAIT并打印警告信息  
net.ipv4.tcp_max_syn_backlog = 4096  
#进入SYN队列最大长度，加大队列长度可容纳更多的等待连接  

硬件：
1. 增大物理内存，更换ssd，换cpu

## 什么情景下做分表，什么情景下做分库

* 介绍  
  读写分离：提高数据库读操作的性能  
  水平分表：按一定的规则（如时间、状态、id）把一个表的数据分散到多个表  
  垂直分表：将一个表中不常用或数据长度较大的的字段分到子表中，使用外键做关联  
  分库：将没有业务关联的数据分到不同的数据库，然后放到不同的服务器上  

* 使用场景  
  读写分离:提高数据库的读性能  
  分库、垂直分表：分散系统负载，让原来一台机器做的事，变成几台机器来做  
  水平分表、分区：缩小索引大小,使查找更快

## 悲观锁和乐观锁，应用中的案例，mysql当中怎么实现，java中的实现

* 悲观锁  
  获取到锁再去做业务逻辑（不论读、写）
* 乐观锁  
  获取数据的时候不需要锁，当更新数据的时候根据版本更新，如果版本不对就表示事务失败

【MySQL】
悲观锁：
begin transaction
select data from table where filed='xxx' for update
update ...
insert ...
commit transaction

乐观锁：
select data,verion from table where filed='xxx'
update table set data=new_data,version =new_version where filed='xxx'
如果更新操作的数据行数大于0表示事务成功。否则失败

## MySQL 索引结构解释一下？（B+ 树）

## 如何分析“慢查询”日志进行 SQL/索引 优化？

## MySQL数据库主从同步怎么实现？

* 原理：
1. 数据库有个bin-log二进制文件，记录了所有sql语句。
2. 我们的目标就是把主数据库的bin-log文件的sql语句复制过来。
3. 让其在从数据的relay-log重做日志文件中再执行一次这些sql语句即可。
4. 下面的主从配置就是围绕这个原理配置
5. 具体需要三个线程来操作：
    * binlog输出线程:每当有从库连接到主库的时候，主库都会创建一个线程然后发送binlog内容到从库。在从库里，当复制开始的时候，从库就会创建两个线程进行处理：
    * 从库I/O线程:当START SLAVE语句在从库开始执行之后，从库创建一个I/O线程，该线程连接到主库并请求主库发送binlog里面的更新记录到从库上。从库I/O线程读取主库的binlog输出线程发送的更新并拷贝这些更新到本地文件，         其中包括relay log文件。
    * 从库的SQL线程:从库创建一个SQL线程，这个线程读取从库I/O线程写到relay log的更新事件并执行。

可以知道，对于每一个主从复制的连接，都有三个线程。拥有多个从库的主库为每一个连接到主库的从库创建一个binlog输出线程，每一个从库都有它自己的I/O线程和SQL线程。

* 同步过程：
1. 主库db的更新事件(update、insert、delete)被写到binlog
2. 从库发起连接，连接到主库
3. 此时主库创建一个binlog dump thread线程，把binlog的内容发送到从库
4. 从库启动之后，创建一个I/O线程，读取主库传过来的binlog内容并写入到relay log.
5. 还会创建一个SQL线程，从relay log里面读取内容，从Exec_Master_Log_Pos位置开始执行读取到的更新事件，将更新内容写入到slave的db.
