不同的引擎对于自增值的保存策略不同。

- MyISAM引擎的自增值保存在数据文件中。
- InnoDB引擎的自增值，其实是保存在了内存里，并且到了MySQL 8.0版本后，才有了“自增值持久化”的能力，也就是才实现了“如果发生重启，表的自增值可以恢复为MySQL重启前的值”，具体情况是：
  - 在MySQL 5.7及之前的版本，自增值保存在内存里，并没有持久化。每次重启后，第一次打开表的时候，都会去找自增值的最大值max(id)，然后将max(id)+1作为这个表当前的自增值。﻿
    举例来说，如果一个表当前数据行里最大的id是10，AUTO_INCREMENT=11。这时候，我们删除id=10的行，AUTO_INCREMENT还是11。但如果马上重启实例，重启后这个表的AUTO_INCREMENT就会变成10。﻿
    也就是说，MySQL重启可能会修改一个表的AUTO_INCREMENT的值。
  - 在MySQL 8.0版本，将自增值的变更记录在了redo log中，重启的时候依靠redo log恢复重启之前的值。

在MySQL里面，如果字段id被定义为AUTO_INCREMENT，在插入一行数据的时候，自增值的行为如下：

1. 如果插入数据时id字段指定为0、null 或未指定值，那么就把这个表当前的 AUTO_INCREMENT值填到自增字段；
2. 如果插入数据时id字段指定了具体的值，就直接使用语句里指定的值。

根据要插入的值和当前自增值的大小关系，自增值的变更结果也会有所不同。假设，某次要插入的值是X，当前的自增值是Y。

1. 如果X<Y，那么这个表的自增值不变；
2. 如果X≥Y，就需要把当前自增值修改为新的自增值。

自增不连续的情况

1. 插入指定非连续主键
2. 插入失败或者事务回滚

为什么Mysql不严格控制连续性？

如果要严格控制只能扩大自增锁的同步范围至提交事务，显然性能不能接受。

注意int(M)中M表示展示的宽度，不影响存储的长度。

表定义的自增值达到上限后的逻辑是：再申请下一个id时，得到的值保持不变。再试图执行插入语句，报主键冲突错误。232-1（4294967295）不是一个特别大的数，对于一个频繁插入删除数据的表来说，是可能会被用完的。因此在建表的时候你需要考察你的表是否有可能达到这个上限，如果有可能，就应该创建成8个字节的bigint unsigned。