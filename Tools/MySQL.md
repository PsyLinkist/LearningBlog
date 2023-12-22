## `os.Getenv("<Env-Var-name>")`
Get the value of Environment Variable from our PC.
Note: It is a method for safety insurance.

## Operations
### import `.sql` file
- workbench:
   Server -> Data Import

### export `.sql` file
- workbench:
   Server -> Data Export

## table Operations
### Create

### Update
```sql
UPDATE your_table
SET your_column1 = 0, your_column2 = 0
WHERE your_condition; -- 如果需要指定更新的特定行，可以添加适当的条件
```

### Insert
```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

### Delete
```sql
DELETE FROM table_name WHERE condition;
```

### Modify
```sql
ALTER TABLE table_name
ADD COLUMN column_name datatype;
```
```sql
ALTER TABLE table_name
DROP COLUMN column_name;
```
```sql
ALTER TABLE table_name
MODIFY COLUMN column_name datatype;
```
