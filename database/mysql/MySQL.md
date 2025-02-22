# MySQL

## 索引失效
* 不满足最左匹配原则
* 索引列计算
* 使用函数处理索引列
* 参数和索引类型不同
* 模糊查询
* or
* not in/not exists
* 索引成本大于全表查询

## show profile
```sql
show profiles; -- 查询select的信息->获取sql语句的query_id
show profile type1,type2... for query_id; -- 查询对应sql的profile信息
```
### type类型
* all：显示所有的性能开销信息
* block io：显示块 IO 相关的开销信息
* context switches: 上下文切换相关开销
* cpu：显示 CPU 相关的信息
* ipc：显示发送和接收相关的开销信息
* memory：显示内存相关的开销信息
* page faults：显示页面错误相关开销信息
* source：显示和 Source_function、Source_file、Source_line 相关的开销信息
* swaps：显示交换次数的相关信息

### 输出
* starting - 正在初始化查询。
* checking permissions - 检查用户权限。
* Opening tables - 打开所需的表。
* init - 初始化查询环境。
* System lock - 系统级别的锁定。
* Table lock - 表级别的锁定。
* executing - 执行查询。
* copy to tmp table - 将数据复制到临时表。
* converting HEAP to MyISAM - 查询结果太大，使用磁盘存储。
* Copying to tmp table on disk - 内存表太大，使用磁盘存储(tmp_table_size参数)。
* Sorting result - 对结果进行排序。
* Create sorting keys - 临时表进行排序。
* Sending data - 发送数据到客户端，数据量大时需要关注。
* statistic - 收集统计信息。
* end - 查询执行结束。


## Unicode、UTF-8、UTF-16 和 UTF-32 
Unicode定义了字符规则，UTF-8、UTF-16 和 UTF-32是Unicode标准的实现。
1. **Unicode**:
   - Unicode 是一个国际标准（ISO/IEC 10646），旨在为世界上几乎所有的字符和文本符号提供一个唯一的数字标识（即码点）。
   - Unicode 本身不是一种编码形式，而是一个字符集，定义了字符的编码范围。

2. **UTF-8**:
   - UTF-8（Unicode Transformation Format, 8-bit）是一种将 Unicode 字符编码转换为 8 位字节序列的格式。
   - UTF-8 是变长编码，使用 1 到 4 个字节表示一个 Unicode 字符，具体长度取决于字符的码点范围。
   - UTF-8 是向后兼容 ASCII 的，因为它的首字节与 ASCII 相同。

3. **UTF-16**:
   - UTF-16（Unicode Transformation Format, 16-bit）是一种将 Unicode 字符编码转换为 16 位字节序列的格式。
   - UTF-16 使用 2 个字节表示基本多文种平面（BMP）中的字符，使用 4 个字节（一对代理项）表示辅助平面上的字符。

4. **UTF-32**:
   - UTF-32（Unicode Transformation Format, 32-bit）是一种将 Unicode 字符编码转换为 32 位字节序列的格式。
   - UTF-32 使用固定长度的 4 个字节表示所有 Unicode 字符，包括辅助平面上的字符。

5. **选择编码方式**:
   - 选择哪种编码方式取决于应用场景和需求。UTF-8 是最常用的编码方式，因为它具有很好的兼容性和效率。
   - UTF-16 在处理包含大量非 BMP 字符的文本时可能更有效率，例如某些亚洲语言。
   - UTF-32 由于固定长度的特性，在某些编程环境中可能更方便处理，尽管它通常不是最优选择，因为它会占用更多的存储空间。

6. **性能和存储**:
   - UTF-8 在存储欧洲语言时通常更加高效，因为它使用更少的字节。
   - UTF-16 在存储亚洲语言时可能更高效，因为大多数字符可以直接用 2 个字节表示。
   - UTF-32 由于固定长度，处理起来可能更简单，但会占用更多的存储空间。

## 字符编码
### 字符集
```sql
show charset;
show charset like '%utf%';
```
### 场景编码
1. utf8/uft8mb3
最长3字节，可以支持大部分语言基本字符，不包含emoji表情。

2. uft8mb4
最长4个字节，几乎包含所有字符。

3. ascii
128个字符

4. gbk/gb2312
gbk是对gb2312的扩展

5. 

### 排序规则
```sql
show collation;
show collation like '%utf%';
```
示例：
utf8_general_ci，字符集_语言_是否区分语言重音、大小写
* ci-case insensitive 不区分大小写
* cs-case sensitive 区分大小写
* ai-accent insensitive 不区分重音
* as-accent sensitive 区分重音
* _bin 按二进制方式比较字符串，区分大小写和重音符号。
[一文搞懂字符集！ （详解utf8 、utf8mb3 和 utf8mb4 的区别，utf8mb4_unicode_ci 和 utf8mb4_general_ci的区别）](https://blog.csdn.net/Lo_CoCo_vE/article/details/132084412)   
[很多人不懂的MySQL编码问题！](https://zhuanlan.zhihu.com/p/424868844)

### 级别
1. 服务器级别
2. 数据库级别
3. 表级别
4. 列级别

### 数据编码解码过程
![93](./image/93.png)

如果是增删改操作，流程为：客户端--->character_set_client--->character_set_connection---->DB

如果是查操作，客户端--->character_set_client--->character_set_connection---->DB---->character_set_result
[五分钟看懂 MySQL 编解码原理](https://www.51cto.com/article/689254.html)
## Cpu飙升排查
本质上是任务过多导致
### 方向
* 慢查询
* 锁等待
* 业务流量增大，需要增加机器资源
* 

### 步骤
#### 问题定位
1. 查看全局状态
mysql记录了统计数据用于排查问题。
```sql
SHOW GLOBAL STATUS LIKE '_';
-- Threads_running 线程数。如果 Threads_running 较高，而 Threads_connected 较低，可能表明存在某些长时间运行的查询，或者可能是由于连接池配置不当导致连接被频繁创建和销毁。
-- Threads_connected 连接数
-- Key_reads：表示从磁盘读取索引块的次数。高的 Key_reads 可能暗示着索引未能完全放入内存中，需要调整 key_buffer_size 参数
-- Key_writes：表示向磁盘写入索引块的次数。频繁的 Key_writes 可能表明索引的写入操作较为频繁，需要考虑索引的优化。
-- Created_tmp_disk_tables 表示在磁盘上创建的临时表的数量。过多的磁盘临时表可能表明某些查询需要优化，或者 tmp_table_size 参数设置过小。

SHOW ENGINE INNODB STATUS;
-- Innodb_row_lock_current_waits：表示当前正在等待的行锁数量。
-- Innodb_deadlocks：显示发生的死锁次数。
-- 

```

2. 查看运行任务
使用show processlist查看正在执行的任务，输出字段如下：
* Id：线程的唯一标识符，是一个数字。

* User：执行线程的 MySQL 用户名。

* Host：用户连接的主机名和端口号，格式通常为 hostname:port。

* db：当前线程所使用的数据库。如果一个线程没有使用任何数据库，这一列将显示为 NULL。

* Command：线程正在执行的命令类型，如 Sleep、Query、Connect 等。

* Time：当前命令已经执行的秒数。对于 Sleep 状态的线程，这表示线程已经空闲了多久。

* State：线程的状态，提供了关于线程当前活动的额外信息。
MySQL的`State`列在`SHOW PROCESSLIST;`命令的输出中提供了当前进程的状态信息。以下是一些常见的`State`值及其含义：

    - **Sleeping**: 线程当前没有执行任何操作，处于空闲状态，等待新的客户端命令。
    - **Query**: 线程正在执行一个查询命令。
    - **Locked**: 线程正在等待行锁或表锁的释放，可能是因为其他进程正在访问该资源。
    - **Copying to tmp table**: 线程正在将数据复制到临时表中，这通常发生在使用`GROUP BY`或`ORDER BY`子句时。
    - **Sorting result**: 线程正在对查询结果进行排序，可能是为了满足`ORDER BY`子句的要求。
    - **Sending data**: 线程正在向客户端发送查询结果。
    - **Creating tmp table**: 线程正在创建一个临时表，这可能发生在某些查询操作中。
    - **Removing duplicates**: 线程正在删除查询结果中的重复行，通常与`DISTINCT`操作相关。
    - **Copying to group table**: 线程正在将数据复制到聚合表中，这通常发生在`GROUP BY`操作中。
    - **Fetching rows**: 线程正在从服务器获取数据行，这可能是查询的一部分。
    - **Altering table**: 线程正在修改表结构。
    - **Rebuilding index**: 线程正在重建索引。
    - **Committing**: 线程正在提交一个事务，将更改永久保存到数据库。
    - **Rolling back**: 线程正在回滚一个事务，撤销所有更改。

* Info：正在执行的 SQL 语句（可能会因为安全或长度原因被截断）。对于 NULL 值，表示该线程没有正在执行的查询，或者查询已经被重置。

* Rows_sent：该线程发送的行数。

* Rows_examined：该线程检查的行数。

* Rows_affected：受该线程当前SQL语句影响的行数（不包括 SELECT 语句）。

结合State和Time大致判断出问题：
* 大量Query+Time时间长->慢查询
* Locked+Time时间长->锁冲突
* Copying to tmp table/Sorting result/Creating tmp table/Removing duplicates->去重/排序等耗时操作
* 都是正常的，那可能是参数配置不合理


####  问题解决
### 慢查询


### 锁等待
* 查看当前锁信息
高版本锁表：performance_schema.data_locks/performance_schema.data_lock_wait
低版本:information_schema.innodb_locks/information_schema.innodb_lock_wait
<img src=".\image\68.jpg" alt="68" />

* 死锁
因为mysql可以检查并中断死锁，所以表并不能捕捉到，需要通过查看日志定位。
1. show engine innodb status->记录了最近一次死锁信息
2. 错误日志
3. 开启innodb_print_all_deadlocks参数，打印详细死锁信息

## Wait-for group
MySQL 中的 InnoDB 存储引擎使用一种称为 "wait-for graph"（等待图）的算法来检测死锁。以下是该算法的工作原理：

1. **等待图**：InnoDB 存储引擎维护一个等待图，图中的节点代表事务，边代表事务之间的锁等待关系。如果事务 A 正在等待事务 B 释放锁，那么就在 A 和 B 之间画一条边。

2. **循环检测**：如果在等待图中存在一个循环，即从一个事务出发，沿着边走一圈后能回到原事务，这表示发生了死锁。例如，事务 A 等待 B，事务 B 等待 C，事务 C 又等待 A。

3. **回滚**：一旦检测到死锁，InnoDB 将选择其中一个或多个事务进行回滚，以打破死锁循环。被回滚的事务会收到一个死锁错误，并且其他事务可以继续执行。

4. **死锁权重**：在选择回滚哪个事务时，InnoDB 会考虑多个因素，如事务的等待时间、事务的大小、回滚的影响等。InnoDB 试图选择一个“成本”最小的事务进行回滚。

5. **锁超时**：InnoDB 还为锁请求设置了超时时间。如果一个事务在超时时间内没有获得所需的锁，它将自动回滚，释放所有锁，并可以重新尝试执行。

6. **锁升级**：InnoDB 通常首先授予行级锁，但如果需要，它可以升级到表级锁。死锁检测算法也适用于锁升级的情况。

7. **死锁日志**：InnoDB 记录死锁发生的情况，包括涉及的事务和锁的信息。这些信息可以用于调试和优化数据库性能。

8. **死锁检测频率**：InnoDB 不是在每次事务请求锁时都检查死锁，而是在某些情况下（如锁请求等待时）才进行死锁检测。


## 事务表
在 MySQL 中，`innodb_trx` 表是 `information_schema` 数据库的一部分，它提供了当前正在进行的所有 InnoDB 事务的详细信息。这个表对于数据库管理员在监控和诊断事务相关问题时非常有用。下面是 `innodb_trx` 表中各个字段的详细解析：

### 字段列表及其描述

1. **trx_id**:
   - 类型: `VARCHAR(18)`
   - 描述: 事务的唯一标识符。这是一个内部生成的序列号，用于唯一标识当前实例中的事务。

2. **trx_state**:
   - 类型: `VARCHAR(13)`
   - 描述: 事务的当前状态。可能的值包括 'RUNNING', 'LOCK WAIT', 'ROLLING BACK' 和 'COMMITTING'。

3. **trx_started**:
   - 类型: `DATETIME`
   - 描述: 事务开始的时间。

4. **trx_requested_lock_id**:
   - 类型: `VARCHAR(81)`
   - 描述: 如果事务当前正在等待锁，这个字段显示等待的锁的标识符。

5. **trx_wait_started**:
   - 类型: `DATETIME`
   - 描述: 事务开始等待锁的时间。如果事务没有在等待锁，该字段为 NULL。

6. **trx_weight**:
   - 类型: `BIGINT`
   - 描述: 事务的权重，表示事务对资源的占用程度。权重越大，事务涉及的行更多，或者事务持有更多的锁。

7. **trx_mysql_thread_id**:
   - 类型: `BIGINT`
   - 描述: 执行事务的 MySQL 线程的 ID。

8. **trx_query**:
   - 类型: `LONGTEXT`
   - 描述: 事务中当前正在执行的 SQL 查询。如果没有查询正在执行，则为 NULL。

9. **trx_operation_state**:
   - 类型: `VARCHAR(64)`
   - 描述: 当前正在执行的事务操作的状态。这可以提供关于事务当前活动的更多细节。

10. **trx_tables_in_use**:
    - 类型: `BIGINT`
    - 描述: 当前在事务中使用的表的数量。

11. **trx_tables_locked**:
    - 类型: `BIGINT`
    - 描述: 当前在事务中锁定的表的数量。

12. **trx_lock_structs**:
    - 类型: `BIGINT`
    - 描述: 事务已经分配的锁结构的数量。

13. **trx_lock_memory_bytes**:
    - 类型: `BIGINT`
    - 描述: 事务分配给锁的内存量（以字节为单位）。

14. **trx_rows_locked**:
    - 类型: `BIGINT`
    - 描述: 事务当前锁定的行数。

15. **trx_rows_modified**:
    - 类型: `BIGINT`
    - 描述: 事务开始以来修改的行数。

16. **trx_concurrency_tickets**:
    - 类型: `BIGINT`
    - 描述: 事务在进入 InnoDB 内部队列之前可以进行的操作数。

17. **trx_isolation_level**:
    - 类型: `VARCHAR(16)`
    - 描述: 事务的隔离级别，如 'READ UNCOMMITTED', 'READ COMMITTED', 'REPEATABLE READ', 或 'SERIALIZABLE'。

18. **trx_unique_checks**:
    - 类型: `INT`
    - 描述: 是否启用了唯一性检查。

19. **trx_foreign_key_checks**:
    - 类型: `INT`
    - 描述: 是否启用了外键约束检查。

20. **trx_last_foreign_key_error**:
    - 类型: `VARCHAR(256)`
    - 描述: 最后一次外键错误的描述（如果有）。

21. **trx_adaptive_hash_latched**:
    - 类型: `INT`
    - 描述: 是否在自适应哈希索引上设置了闩锁。

22. **trx_adaptive_hash_timeout**:
    - 类型: `INT`
    - 描述: 自适应哈希索引的超时设置。

## 索引压缩

### mysiam
pack_keys 是一个与MyISAM存储引擎相关的表选项，用于控制MyISAM表的索引键值是否应该被压缩。

#### `pack_keys` 的设置选项

`pack_keys` 参数可以有以下几种设置：

- `pack_keys=0`：禁用键压缩。这是默认设置，意味着索引键不会被压缩。
- `pack_keys=1`：启用键压缩。在这种模式下，MyISAM会尝试压缩所有索引键，除了那些用作行指针的部分。
- `pack_keys=DEFAULT`：这将只压缩字符串类型的键（CHAR, VARCHAR, TEXT），而不压缩数值类型的键（如INT, DECIMAL等）。

#### 工作原理

当`pack_keys`设置为压缩时（`pack_keys=1`或`DEFAULT`），对于字符串类型的键，MyISAM通过只存储字符串的变化部分来实现压缩。这类似于前缀压缩，其中只记录与前一个键值不同的部分。

#### 性能影响

- **空间节省**：启用`pack_keys`可以减少索引的存储空间，这对于磁盘空间有限的系统可能非常有用。
- **性能**：因为每个值都依赖前面的值，所以查找的时候无法使用二分法只能从头开始扫描，还有倒序扫描速度下降，因此等值查询平均都需要扫描半个索引块。

#### 使用场景

- **大量文本数据**：对于包含大量文本字段的表，启用`pack_keys`可能会显著减少索引的大小。
- **读密集型应用**：如果应用主要是读取操作，并且磁盘I/O是性能瓶颈，压缩索引可能有助于提高性能。
- **空间敏感型应用**：在磁盘空间非常宝贵的环境中，压缩索引可以是一个有价值的优化。

### 设置方法

在创建或修改MyISAM表时，可以通过SQL语句设置`pack_keys`：

```sql
CREATE TABLE my_table (
    id INT,
    data VARCHAR(255),
    INDEX data_index (data)
) ENGINE=MyISAM PACK_KEYS=1;

ALTER TABLE my_table PACK_KEYS=0;
```

### innodb
在InnoDB存储引擎中，使用`ROW_FORMAT=COMPRESSED`选项可以启用表数据和索引的压缩。这种压缩方式旨在减少数据在磁盘上的存储空间，从而可能减少I/O操作，提高数据加载的速度。下面详细解析`ROW_FORMAT=COMPRESSED`是如何实现压缩的：

#### 压缩机制

1. **页压缩**：
   - InnoDB存储数据和索引在磁盘上的基本单位是“页”，默认大小为16KB。
   - 当表设置为`COMPRESSED`行格式时，InnoDB会尝试将这些16KB的页压缩到更小的页中，如8KB、4KB或更小，具体取决于`KEY_BLOCK_SIZE`的设置。
   - 压缩使用的是zlib库，这是一个广泛使用的数据压缩库。

2. **压缩过程**：
   - 当数据页（包括索引页）首次写入磁盘前，InnoDB会尝试将其压缩到指定的`KEY_BLOCK_SIZE`。
   - 如果压缩成功，压缩后的页将被写入磁盘。如果压缩失败（即数据无法压缩到小于或等于指定的`KEY_BLOCK_SIZE`），则原始的16KB页将被写入磁盘。

3. **压缩页的管理**：
   - 每个压缩页在内存中都有一个对应的未压缩的16KB的“修改缓冲区”页。
   - 当压缩页被读取进内存时，它会被解压缩回16KB的格式，以便进行处理。
   - 当对页进行修改时，修改首先应用于内存中的未压缩页。之后，在将页写回磁盘之前，页将再次被压缩。

#### 性能和存储考虑

- **存储效率**：使用`ROW_FORMAT=COMPRESSED`可以显著减少数据和索引占用的磁盘空间，特别是对于包含大量可压缩数据（如文本或冗余数据）的表。
- **CPU开销**：压缩和解压缩数据需要额外的CPU资源，这可能会影响到系统的CPU性能，特别是在高负载情况下。
- **I/O性能**：虽然压缩减少了磁盘上的数据量，从而可能减少了读取数据所需的I/O操作，但频繁的压缩和解压缩可能会抵消这一优势。

#### 使用场景

- **适用于读多写少的场景**：如果应用主要进行读操作，压缩可以减少读取数据所需的I/O，提高性能。
- **数据冷存储**：对于不经常访问的数据，使用压缩可以节省存储成本。

## hint
用于向数据库查询优化器提供额外的信息，以影响查询执行计划的选择。通过使用hint，开发者可以在不修改数据库本身统计信息和结构的情况下，指导优化器采取特定的行动。这可以帮助优化复杂的查询，特别是在优化器未能选择最优查询计划的情况下。
### 索引选择
* USE INDEX：指定查询应该使用的索引。
* FORCE INDEX：强制查询使用特定的索引，即使MySQL认为不使用索引或使用其他索引更优。
* IGNORE INDEX：告诉优化器忽略一个或多个索引。

### 查询执行方式
* STRAIGHT_JOIN：强制MySQL按照FROM子句中表的顺序来连接表。
* SQL_SMALL_RESULT：指示优化器查询结果集较小，优化器可以选择更适合处理小结果集的查询计划。
* SQL_BIG_RESULT：指示优化器查询结果集较大，优化器可能选择不同的查询计划以处理大量数据。

### 其他
* SQL_NO_CACHE：指示MySQL不要将查询结果缓存，这对于一次性查询很有用。
* SQL_CALC_FOUND_ROWS：告诉MySQL计算符合条件的总行数，即使查询本身有LIMIT限制。

##  MySQL单表数据达到什么程度需要分表分库
MYSQL单表数据达2000万性能严重下降，经验得出大部分单条数据的大小在1k左右，所以一个页可以存储的数据大约为16条。Innodb使用B+树来进行索引存储，主键bigint占8字节，指针占6字节，那么根节点可以存储主键+指针数就是16*1024/14 = 1170.
两层索引可以存储16*1170=18720条数据
三层索引可以存储16*1170*1170 = 21902400 ～约等于2000w条数据。
所以超过2000w条数据之后理论上就需要四层索引来处理，那么为什么超过之后性能会下降严重呢？
* 因为读取一次索引就是一次IO操作，那么为什么四次IO就会比三次IO要性能下降很多呢？
* 因为InnoDB使用innodb_buffer_pool_size来设置缓存，会存储索引和数据，一般innodb_buffer_pool_size的大小默认为128M，大概可以存储2层缓存，所以一般三层索引只需要执行一次IO即可，四层索引需要执行2次IO，性能才会严重下降。
所以在数据量比较大的情况下，可以通过分表来降低单表的存储数据，来加快数据的查询速度

## 排名内置函数

`8.0以上版本使用`

* RANK()  
 并列跳跃排名，并列即相同的值，相同的值保留重复名次，遇到下一个不同值时，跳跃到总共的排名。 1 2 2 4 5  
* DENSE_RANK()
并列连续排序，并列即相同的值，相同的值保留重复名次，遇到下一个不同值时，依然按照连续数字排名。 1 2 2 3 4
* ROW_NUMBER()
连续排名，即使相同的值，依旧按照连续数字进行排名。 1 2 3 4

```sql
// 需要配合 over 使用，partition by分区，order by是排序
select a,b,rank() over(partition by a order by a desc) as 'rank' from table_name
```

## 日期函数

* date_add()
* date_sub()

```sql
set @dt = now();

select date_add(@dt, interval 1 day); -- add 1 day
select date_add(@dt, interval 1 hour); -- add 1 hour
select date_add(@dt, interval 1 minute); -- ...
select date_add(@dt, interval 1 second);
select date_add(@dt, interval 1 microsecond);
select date_add(@dt, interval 1 week);
select date_add(@dt, interval 1 month);
select date_add(@dt, interval 1 quarter);
select date_add(@dt, interval 1 year);

select date_add(@dt, interval -1 day); -- sub 1 day

 select date_add(@dt, interval '01:15:30' hour_second);
 select date_add(@dt, interval '1 01:15:30' day_second);
```

* datediff(date1,date2)

```sql
select datediff('2008-08-08', '2008-08-01'); -- 7
select datediff('2008-08-01', '2008-08-08'); -- -7
```

## 精度函数

* round(number,num_digits)四舍五入

```sql
    number：需要四舍五入的数字
    num_digits：按此位数对 number 参数进行四舍五入
    0 表示取整
    1 表示保留一位小数，2 表示保留两位小数，依此类推
    -1 表示十位数取整，-2 表示百位数取整，依次类推

    SELECT ROUND(100.3465,2),ROUND(100,2),ROUND(0.6,2),ROUND(114.6,-1);

    100.35,100，0.6,110
```

## mysql并发数高

大多数情况下，开发会使用多线程并行处理方法加快任务进度，减少DB执行时间，但是很多人都错估了DB承受能力导致mysql经常有running high情况，而running high 服务器最显著的表示CPU load、idle 异常，并发数设置过高并不能给程序带来任何收益，反而恰恰相反，增加了执行时间。

并发数高影响：

      1.并发线程数太高，线程调度会严重消耗资源，当运行线程数超过32个（innodb_thread_concurrency）时，请求线程会在mysql innodb层等待，处于waiting for innodb queue状态，同时线程会有大量上下文切换，消耗CPU。

       2.不同线程访问相同资源，会出现资源竞争或者mysql事务锁，从而出现SQL拥塞现象。mysql中会有mutex来保护共享资源，当有线程需要使用这些共享资源时，会请求获得mutex量，请求成功的线程进入临界区，而请求失败的线程只能等待它释放这个mutex，所以并发线程越多，mutex的竞争越激烈；当某些并发线程访问更新一表中数据时，而恰巧mysql 隔离级别是REPEATABLE READ，会有间隙锁，线程高，锁竞争多，SQL拥塞，执行时间较长。

资源是有限，开启过多进程并不会提升性能，反而会下降。在满足业务要求下，尽可能降低资源消耗，需求时间和性能资源必须有取舍，取舍优先级必须性能资源大于时间消耗。

在确定任务完成时间需求，开发合理分解任务后，可以这样计算并发数：

     情况一、业务要求时间是T1，单线程运行完是T2，T1>T2，那单线程运行就额可以满足要求，没有必要采用多线程并发处理；

     情况二、如果T2>T1,取N=T2/T1,以这个为基准，测试N个并发下运行时间是否小于T1，如果N个并发还不能满足业务时间要求，则加大一个并发进行再次测试，找到一个满足业务需求的最小并发数。

## mysql从库延迟

mysql db从库延迟一直是困扰着我们noc dba，几乎每天都会有某个域的延迟告警，大多数情况下，延迟原因为以下两种：   1.批量数据增删改 2.大表加索引操作3.表无主键，一次更新多条记录，对于某些域DB而言，开发可能认为从库没业务延迟可以忽略，但是无规则不成方圆，只有我们按照规范约束数据库行为才能提供更好运维环境。

1.批量数据增删改操作可以拆分，按照每次处理1000条，sleep 0.3s模式循环操作，目前我们线上mysql DB都是5.5.44版本，复制操作是单线程，普通情况一个单线程每秒最大操作数据行数在8k-10k行左右

2.对于BI系统而言为了谋求插入速度，常常会对表先insert再创建索引，几百万数据表创建索引延迟数据是非常可观的，所以先创索引再插入，保证线上DB稳定。

3.表无主键，一次操作100+条记录时，而这条SQL是全表扫描/低效的索引扫描，从库复制时会产生延迟，因为我们线上DB是采用row模式binlog复制，主库更新100条记录，从库就需执行100次update语句，而每个SQL效率极其低下。

## mysql服务端/客户端通信协议

半双工 只有一端向另一端发送数据，只有数据全部接受完才能响应(max_allowed_packet客户端和服务端每次通讯最大数据包大小)

* TCP/IP

* 命名管道和共享内存(仅适合单机)

1. 命名管道
服务器 --enable-named-pipe  客户端 --protocol=pipe
2. 共享内存
服务器 --shared-memory 客户端 --protocol=memory

* Unix域套接字(仅适合unix单机)  

* 为什么mysql通信协议要选用半双工的形式呢？

1. 实现简单，容易维护
2. 数据库查询的机制是一个sql请求一个数据回复，半双工就可以满足要求，数据库的性能瓶颈是在sql的执行时间，和数据库的底层实现等有关，和通讯机制关系不大。(个人理解)

## mysql无法在同一个sql中同时对一张表进行查询和更新

```sql
update table_name set a = (select b from table_name where id = 1);  -- 执行报错
-- 可以通过临时表的方式绕过限制(查询被当做临时表)
update table_name inner join (select b from table_name where id = 1) table_name1 on c set table_name.a = table_name1.b;
```

## on where having的区别

* on 是在多表连接作为条件去判断两条记录是否匹配的条件
* where 是在返回结果之前对数据进行过滤的条件，不可以使用聚合函数，可以使用所有列
* having 一般搭配 group by 使用(也可以单独使用)，对返回的数据集进行每一个组的过滤，可以使用聚合函数，因为是对返回的集合的操作，所以只能操作数据集中有的列
* 执行顺序 on -> where -> having

## BufferPool缓冲池

<https://blog.csdn.net/wuhenyouyuyouyu/article/details/93377605>  
<https://blog.csdn.net/m0_37892044/article/details/121795586>

## MVCC

<https://blog.csdn.net/qq_38538733/article/details/88902979>

## 行数据的最大值

一条记录占用的最大存储空间是有限制的，除了 BLOB 或者 TEXT 类型的列之外(根据具体行格式判断，Compact存放前768字节，Dynamic等不存放)，其他所有的列（不包括隐藏列和记录头信息）占用的字节长度加起来不能超过 65535 个字节(数据页大小16k)。
<https://zhuanlan.zhihu.com/p/53413773>

## mysql统计表和实际不符

统计表采用定期同步的方式去落盘数据，会发生统计表(information_schema.tables/show table status等)和实际情况不符的情况。

* 解决方法：将时间设置为实时更新

```sql
-- 全局设置实时更新
SET GLOBAL information_schema_stats_expiry=0;
SET @@GLOBAL.information_schema_stats_expiry=0;

-- 会话实时更新
SET SESSION information_schema_stats_expiry=0;
SET @@SESSION.information_schema_stats_expiry=0;
```

## 删除数据的方式

* delete from table_name

1. delete 是 DML 语句，只删除数据不删除结构，通过一行一行删除并且记录事务日志，可以回滚。
2. 执行会触发trigger
3. 删除数据后innodb和myisam都不会立刻释放空间，只会标记记录为删除状态，删除的空间可重用(可使用optmize table table_name 整理空间碎片)。
    * myisam
    删除前
    <img src=".\image\21.jpg" alt="21" />
    删除后
    <img src=".\image\22.jpg" alt="22" />
    可以看出虽然行数和平均的行数长度都为0，但是数据的长度没有改变，只是作为空间碎片的数据重复使用，并没有释放磁盘空间。
    * innodb
    删除前
    <img src=".\image\23.jpg" alt="23" />
    删除后
    <img src=".\image\24.jpg" alt="24" />
    可以看出虽然行数和平均的行数长度都为0，但是数据的长度没有改变(删除后的空间在innodb引擎不作为空间碎片)。
4. 执行后不重置 auto_increment

* truncate table table_name

1. truncate 是 DDL 语句，速度快(不记录undo日志)，执行后无法回滚。
2. 不触发trigger
3. innodb和myisam引擎都一致，执行后立即释放磁盘空间
4. 执行后重置 auto_increment

* drop table table_name

1. drop 是 DDL 语句
2. 执行后立即释放磁盘空间
3. 执行后删除表的数据、结构、触发器等，依赖于该表的存储过程/函数状态变为 invalid

## 自适应哈希索引(AHI)

自适应哈希索引是mysql内部一种加快查询的优化措施，由innodb引擎自己判断构建，只能针对等值查询。
[MySQL AHI 实现解析](https://cloud.tencent.com/developer/article/1004516)

## 其他索引

## 多维索引

* [空间填充曲线](https://www.cnblogs.com/tgzhu/p/8286616.html)
* 网格文件
* 分段散列
* 多键索引
* kd-树
* 四叉树
* R-树
* 位图索引
[【高级数据库】第二章 第04讲 多维索引](https://blog.csdn.net/qq_36426650/article/details/103324224)

## 行式存储 VS 列式存储 ? [OLTP VS OLAP](https://github.com/Vonng/ddia/blob/master/ch3.md#%E4%BA%8B%E5%8A%A1%E5%A4%84%E7%90%86%E8%BF%98%E6%98%AF%E5%88%86%E6%9E%90)

## 行式存储(OLTP)

* 需要频繁更新和插入数据
* 记录列不多，没有复杂的分析场景

在高层次上，我们看到存储引擎分为两大类：针对 事务处理（OLTP） 优化的存储引擎和针对 在线分析（OLAP） 优化的存储引擎。这两类使用场景的访问模式之间有很大的区别：  

* OLTP 系统通常面向最终用户，这意味着系统可能会收到大量的请求。为了处理负载，应用程序在每个查询中通常只访问少量的记录。应用程序使用某种键来请求记录，存储引擎使用索引来查找所请求的键的数据。硬盘查找时间往往是这里的瓶颈。
* 数据仓库和类似的分析系统会少见一些，因为它们主要由业务分析人员使用，而不是最终用户。它们的查询量要比 OLTP 系统少得多，但通常每个查询开销高昂，需要在短时间内扫描数百万条记录。硬盘带宽（而不是查找时间）往往是瓶颈，列式存储是针对这种工作负载的日益流行的解决方案。

## [列式存储(OLAP)](https://github.com/Vonng/ddia/blob/master/ch3.md#%E4%BA%8B%E5%8A%A1%E5%A4%84%E7%90%86%E8%BF%98%E6%98%AF%E5%88%86%E6%9E%90)

## 优势

* 针对不同数据类型压缩数据
* 不读取无效数据，适用于列非常多的场景
* 提高有效数据读取效率，减少不需要数据的读取
* 对数据按不同纬度排序处理应对不同场景的分析操作+数据备份

## 中间件

* Hbase
* Druid
* Cassandra
* clickhouse
[列存数据库，不只是列式存储](https://cn.kyligence.io/blog/%E5%88%97%E5%AD%98%E6%95%B0%E6%8D%AE%E5%BA%93%EF%BC%8C%E4%B8%8D%E5%8F%AA%E6%98%AF%E5%88%97%E5%BC%8F%E5%AD%98%E5%82%A8/)

## SQL语句执行顺序

<img src=".\image\66.jpg" alt="66" />

## 基本概念

* 聚簇索引
* 索引覆盖
* 索引下推
* MRR
<img src=".\image\87.jpg" alt="87" />
<img src=".\image\88.jpg" alt="88" />

## mysql问题合集

* char和varchar
* datetime和timestamp
* uuid适合作为主键吗？
* 索引原则
* mysql索引数据结果选型
* 为什么用自增id作为主键
* 平衡二叉树和红黑色
* 数据库连接使用后不关闭的弊端？
* 短索引
* SQL注入
* Statement和PrepareStatement
* mysql常见引擎
* 三大范式
<img src=".\image\80.jpg" alt="80" />
<img src=".\image\81.jpg" alt="81" />
<img src=".\image\82.jpg" alt="82" />
<img src=".\image\83.jpg" alt="83" />
<img src=".\image\84.jpg" alt="84" />
<img src=".\image\85.jpg" alt="85" />
<img src=".\image\86.jpg" alt="86" />

## 查询语句执行过程

<img src=".\image\89.jpg" alt="89" />

## 主从同步

<img src=".\image\90.jpg" alt="90" />

## 正排索引和倒排索引

* 正排索引以文档为关键字，倒排索引以字或词为索引

## 重复插入数据

### ON DUPLICATE KEY UPDATE(mysql独有语法)

插入数据时，如果表中存在对应主键数据，会执行 ON DUPLICATE KEY UPDATE后续更新操作，若后续字段有发生更新，返回2(用于区分不同情况)，若覆盖更新的字段和原字段数据一致，返回0；如果表中没有该主键，插入数据返回1。

```sql
-- 需要使用主键判断
INSERT INTO table_name (`id`, `create_time`, `update_time`) VALUES (2, 0, 0) on DUPLICATE KEY UPDATE create_time=2 , update_time=1;
```

[ON DUPLICATE KEY UPDATE 用法与说明](https://blog.csdn.net/zyb2017/article/details/78449910)

### REPLACE

插入数据时，如果表中存在对应主键数据，会先删除记录，再插入。
[MySQL replace语句](https://www.yiibai.com/mysql/replace.html)

## GTID(Global Transaction Identifier)

事务全局标识符，分为两部分：服务器UUID唯一标识符+ID，在row或者mixed模式生效，事务提交后会写入binlog递增生成ID，用于全局(多服务器)唯一表示事务。在单源头服务器中，gtid是递增的，多源头服务器中，因为在同步数据时会按源数据的gtid写入从数据库，所以gtid是乱序的。

### 作用

#### 主从故障切换

* 选取新主
传统模式下通过日志的位置和日志的偏移量(那个文件同步到哪里)衡量从数据库数据同步完整性，这个过程复杂且易错，gtid可以简化这个过程。
* 指定从服务器复制位置
传统模式下需要为每个从服务器设置同步新位置，gtid模式下，只需要同步executed_gtid_set没有执行过的事务即可。

#### 主从数据一致性

gtid_executed->执行过的gtid，通过取主从服务器集合差集，判断主从数据一致性情况

#### 多机房同步数据回环问题

通过executed_gtid_set判断事务是否执行过。

## NULL

表示数据不存在，意味着这个字段没有值，和类型的空值不相同。

* NULL !=NULL
* 会被汇总函数忽略
* 排序时，根据 SQL_MODE 变量(NULLS_FIRST||NULLS_LAST)排序

## datetime vs timestamp

* datetime需要5-8字节，timestamp需要4字节
* timestamp范围 1970-2027，datetime没有限制
* timestamp使用utc时区格式存储，使用时根据不同时区转换，datetime不跟随时区变化
