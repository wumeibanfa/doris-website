---
{
    "title": "SHOW-ALTER",
    "language": "en"
}
---

## SHOW-ALTER

### Name

SHOW ALTER

### Description

This statement is used to display the execution of various modification tasks currently in progress

```sql
SHOW ALTER [CLUSTER | TABLE [COLUMN | ROLLUP] [FROM db_name]];
```

TABLE COLUMN: show ALTER tasks that modify columns
                      Support syntax [WHERE TableName|CreateTime|FinishTime|State] [ORDER BY] [LIMIT]
        TABLE ROLLUP: Shows the task of creating or deleting a ROLLUP index
        If db_name is not specified, the current default db is used
        CLUSTER: Displays tasks related to cluster operations (only for administrators! To be implemented...)

### Example

1. Display the task execution of all modified columns of the default db

   ```sql
    SHOW ALTER TABLE COLUMN;
   ```

2. Display the task execution status of the last modified column of a table

   ```sql
   SHOW ALTER TABLE COLUMN WHERE TableName = "table1" ORDER BY CreateTime DESC LIMIT 1;
   ```

3. Display the task execution of creating or deleting ROLLUP index for the specified db

   ```sql
   SHOW ALTER TABLE ROLLUP FROM example_db;
   ```

4. Show tasks related to cluster operations (only for administrators! To be implemented...)

   ```
   SHOW ALTER CLUSTER;
   ```

### Keywords

    SHOW, ALTER

### Best Practice

