---
{
    "title": "WEEK_FLOOR",
    "language": "zh-CN"
}
---

## 描述

将日期时间值向下舍入到最接近的指定周间隔。如果提供了起始时间（origin），则使用该时间作为计算间隔的参考点。

## 语法

```sql
WEEK_FLOOR(<datetime>)
WEEK_FLOOR(<datetime>, <origin>)
WEEK_FLOOR(<datetime>, <period>)
WEEK_FLOOR(<datetime>, <period>, <origin>)
```

## 参数

| 参数 | 说明 |
| ---- | ---- |
| `<datetime>` | 要向下舍入的日期时间值，类型为 DATETIME 或 DATETIMEV2 |
| `<period>` | 周间隔值，类型为 INT，表示每个间隔的周数 |
| `<origin>` | 间隔的起始点，类型为 DATETIME 或 DATETIMEV2；默认为 0001-01-01 00:00:00 |

## 返回值

返回类型为 DATETIME，表示向下舍入后的日期时间值。结果的时间部分将被设置为 00:00:00。

**注意：**
- 如果未指定周期，则默认为 1 周间隔。
- 周期必须是正整数。
- 结果总是向下舍入到过去的时间。
- 返回值的时间部分始终设置为 00:00:00。

## 举例

```sql
SELECT WEEK_FLOOR('2023-07-13 22:28:18', 2);
```

```text
+------------------------------------------------------------+
| week_floor(cast('2023-07-13 22:28:18' as DATETIMEV2(0)), 2) |
+------------------------------------------------------------+
| 2023-07-03 00:00:00                                        |
+------------------------------------------------------------+
```
