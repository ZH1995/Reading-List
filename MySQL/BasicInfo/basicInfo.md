## 1.MySQL内部核心组件

### Server层

**连接器：**管理连接和权限校验。

**缓存器：**MySQl5.7的产物，在MySQL8.0之后移出。将SQL语句和查询结果缓存为k-v对，一旦记录发生更新或SQL语句发生变更，将无法使用缓存。在查询表常量或不会变化的数据时比较有用。

**词法分析器：**语法分析、词法分析。

**优化器：**生成执行计划，选择索引。

**执行器：**调用引擎的接口，获取查询结果。

### 引擎层

负责读写磁盘，实现数据的结构化存储。常见的引擎包括InnoDB、MyISAM、Memory等。

## 2.一条SQL语句的执行流程

例如：update name='xxx' from table_name where id=1;

- 查原始数据：从Buffer Pool（缓冲池）查询id=1记录的记录，若不存在则从磁盘文件（ibd文件）加载id=1记录所在的整页数据到Buffer Pool。
- 写undo log：将旧值写入undo log，便于回滚。
- 更新数据：更新内存数据为新值。
- 写redo log：将新值写入Redo Log Buffer。
- 将redo log写入磁盘：按照redo log的顺序，写入磁盘，准备提交事务。
- 写binlog：将新值写入binlog并落盘，准备提交事务。
- 向redo log写入commit标记：响应客户端提交事务完成。
- 异步刷盘：系统空闲时，后台线程将内存中的修改，以页为单位写入磁盘。



**WAL机制**

Write-Ahead Logging，磁盘预写机制。先写redo log，后刷新数据表文件（ibd文件），可以充分利用redo log顺序写的优势，提升写入性能。