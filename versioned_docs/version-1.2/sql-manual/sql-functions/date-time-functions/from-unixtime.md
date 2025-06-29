---
{
    "title": "FROM_UNIXTIME",
    "language": "en"
}
---

## from_unixtime
### description
#### syntax

`DATETIME FROM UNIXTIME (INT unix timestamp [, VARCHAR string format]`

Convert the UNIX timestamp to the corresponding time format of bits, and the format returned is specified by `string_format`

Input is an integer and return is a string type

Support `date_format`'s format, and default is `%Y-%m-%d %H:%i:%s`

### example

```
mysql> select from_unixtime(1196440219);
+---------------------------+
| from_unixtime(1196440219) |
+---------------------------+
| 2007-12-01 00:30:19       |
+---------------------------+

mysql> select from_unixtime(1196440219, '%Y-%m-%d');
+-----------------------------------------+
| from_unixtime(1196440219, '%Y-%m-%d')   |
+-----------------------------------------+
| 2007-12-01                              |
+-----------------------------------------+

mysql> select from_unixtime(1196440219, '%Y-%m-%d %H:%i:%s');
+--------------------------------------------------+
|From unixtime (1196440219,'%Y-%m-%d %H:%i:%s')    |
+--------------------------------------------------+
| 2007-12-01 00:30:19                              |
+--------------------------------------------------+
```

### keywords

    FROM_UNIXTIME,FROM,UNIXTIME
