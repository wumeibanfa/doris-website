---
{
    "title": "主键模型",
    "language": "zh-CN"
}
---

当用户有数据更新需求时，可以选择使用主键数据模型（Unique）。主键模型能够保证 Key（主键）的唯一性，当用户更新一条数据时，新写入的数据会覆盖具有相同 key（主键）的旧数据。

主键模型提供了两种实现方式：

-   读时合并 (Merge-on-Read)。在读时合并实现中，用户在进行数据写入时不会触发任何数据去重相关的操作，所有数据去重的操作都在查询或者 compaction 时进行。因此，读时合并的写入性能较好，查询性能较差，同时内存消耗也较高。

-   写时合并 (Merge-on-Write)。从 1.2 版本中，Doris 引入了写时合并实现，该实现会在数据写入阶段完成所有数据去重的工作，因此能够提供非常好的查询性能。在开启了写时合并选项的 Unique 表上，数据在导入阶段就会去将被覆盖和被更新的数据进行标记删除，同时将新的数据写入新的文件。在查询的时候，所有被标记删除的数据都会在文件级别被过滤掉，读取出来的数据就都是最新的数据，消除掉了读时合并中的数据聚合过程，并且能够在很多情况下支持多种谓词的下推。因此在许多场景都能带来比较大的性能提升，尤其是在有聚合查询的情况下。

下面以一个典型的用户基础信息表，来看看如何建立读时合并和写时合并的主键模型表。这个表数据没有聚合需求，只需保证主键唯一性。（这里的主键为 user_id + username）。

| ColumnName    | Type         | IsKey | Comment      |
| ------------- | ------------ | ----- | ------------ |
| user_id       | BIGINT       | Yes   | 用户 id       |
| username      | VARCHAR(50)  | Yes   | 用户昵称     |
| city          | VARCHAR(20)  | No    | 用户所在城市 |
| age           | SMALLINT     | No    | 用户年龄     |
| sex           | TINYINT      | No    | 用户性别     |
| phone         | LARGEINT     | No    | 用户电话     |
| address       | VARCHAR(500) | No    | 用户住址     |
| register_time | DATETIME     | No    | 用户注册时间 |

## 读时合并

读时合并的建表语句如下：

```sql
CREATE TABLE IF NOT EXISTS example_tbl_unique
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `username` VARCHAR(50) NOT NULL COMMENT "用户昵称",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `phone` LARGEINT COMMENT "用户电话",
    `address` VARCHAR(500) COMMENT "用户地址",
    `register_time` DATETIME COMMENT "用户注册时间"
)
UNIQUE KEY(`user_id`, `username`)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 1
PROPERTIES (
"replication_allocation" = "tag.location.default: 1"
);
```

## 写时合并

写时合并建表语句为：

```sql
CREATE TABLE IF NOT EXISTS example_tbl_unique_merge_on_write
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `username` VARCHAR(50) NOT NULL COMMENT "用户昵称",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `phone` LARGEINT COMMENT "用户电话",
    `address` VARCHAR(500) COMMENT "用户地址",
    `register_time` DATETIME COMMENT "用户注册时间"
)
UNIQUE KEY(`user_id`, `username`)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 1
PROPERTIES (
"replication_allocation" = "tag.location.default: 1",
"enable_unique_key_merge_on_write" = "true"
);
```

用户需要在建表时添加下面的 property 来开启写时合并。

```Plain
"enable_unique_key_merge_on_write" = "true"
```
:::caution 注意
在 2.1 版本中，写时合并将会是主键模型的默认方式。所以如果使用 Doris 2.1 版本，务必要阅读相关建表文档。
:::

## 使用注意

-   Unique 表的实现方式只能在建表时确定，无法通过 schema change 进行修改。

-   旧的 Merge-on-Read 的实现无法无缝升级到 Merge-on-Write 的实现（数据组织方式完全不同），如果需要改为使用写时合并的实现版本，需要手动执行 `insert into unique-mow-table select * from source table` 来重新导入。

-   整行更新：Unique 模型默认的更新语义为整行 `UPSERT`，即 UPDATE OR INSERT，该行数据的 key 如果存在，则进行更新，如果不存在，则进行新数据插入。在整行 `UPSERT` 语义下，即使用户使用 insert into 指定部分列进行写入，Doris 也会在 Planner 中将未提供的列使用 NULL 值或者默认值进行填充

-   部分列更新。如果用户希望更新部分字段，需要使用写时合并实现，并通过特定的参数来开启部分列更新的支持。请查阅文档[部分列更新](../../data-operate/update/update-of-unique-model)获取相关使用建议。