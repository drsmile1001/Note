# SQL

## 查詢各個資料表與欄位

```sql
SELECT   SchemaName = c.table_schema,
         TableName = c.table_name,
         ColumnName = c.column_name,
         DataType = data_type,
         NullAble = (case when c.IS_NULLABLE = 'YES' then 'NULL' else 'NOT NULL' end),
         IsIdentity = COLUMNPROPERTY(object_id(c.TABLE_SCHEMA+'.'+c.TABLE_NAME), c.COLUMN_NAME, 'IsIdentity')
FROM     information_schema.columns c
         INNER JOIN information_schema.tables t
           ON c.table_name = t.table_name
              AND c.table_schema = t.table_schema
              AND t.table_type = 'BASE TABLE'
ORDER BY SchemaName,TableName,ORDINAL_POSITION
```

## 查詢各個資料表與欄位資訊

```sql
SELECT SchemaName = c.table_schema,
    TableName = c.table_name,
    ColumnName = c.column_name,
    DataType = data_type + (CASE WHEN c.CHARACTER_MAXIMUM_LENGTH IS NOT NULL THEN 
        (CASE WHEN c.CHARACTER_MAXIMUM_LENGTH = -1 THEN
             '(max)' 
        ELSE
             '(' + CONVERT(varchar(10), c.CHARACTER_MAXIMUM_LENGTH) + ')' 
        END)
    ELSE 
        ''
    END),
    NullAble = CASE WHEN c.IS_NULLABLE = 'YES' THEN 'NULL' ELSE 'NOT NULL' END,
    IsIdentity = COLUMNPROPERTY(object_id(c.TABLE_SCHEMA+'.'+c.TABLE_NAME), c.COLUMN_NAME, 'IsIdentity'),
    PK = CASE WHEN pk.COLUMN_NAME IS NOT NULL THEN 'PK ' + CONVERT(varchar(10), pk.ORDINAL_POSITION) ELSE '' END,
    FK = CASE WHEN fk.COLUMN_NAME IS NOT NULL THEN 'FK' ELSE '' END
FROM information_schema.columns c
INNER JOIN information_schema.tables t
    ON c.table_name = t.table_name
    AND c.table_schema = t.table_schema
    AND t.table_type = 'BASE TABLE'
LEFT JOIN (
    SELECT
        ku.TABLE_CATALOG,
        ku.TABLE_SCHEMA,
        ku.TABLE_NAME,
        ku.COLUMN_NAME,
        ku.ORDINAL_POSITION
    FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS AS tc
    INNER JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS ku
        ON tc.CONSTRAINT_TYPE = 'PRIMARY KEY'
        AND tc.CONSTRAINT_NAME = ku.CONSTRAINT_NAME
) pk
    ON c.TABLE_CATALOG = pk.TABLE_CATALOG
    AND c.TABLE_SCHEMA = pk.TABLE_SCHEMA
    AND c.TABLE_NAME = pk.TABLE_NAME
    AND c.COLUMN_NAME = pk.COLUMN_NAME
LEFT JOIN(
    SELECT  DISTINCT
        KCU1.TABLE_CATALOG,
        KCU1.TABLE_SCHEMA,
        KCU1.TABLE_NAME,
        KCU1.COLUMN_NAME
    FROM INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS AS RC
    INNER JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS KCU1 
        ON KCU1.CONSTRAINT_CATALOG = RC.CONSTRAINT_CATALOG  
        AND KCU1.CONSTRAINT_SCHEMA = RC.CONSTRAINT_SCHEMA 
        AND KCU1.CONSTRAINT_NAME = RC.CONSTRAINT_NAME 
) fk
    ON c.TABLE_CATALOG = fk.TABLE_CATALOG
    AND c.TABLE_SCHEMA = fk.TABLE_SCHEMA
    AND c.TABLE_NAME = fk.TABLE_NAME
    AND c.COLUMN_NAME = fk.COLUMN_NAME
ORDER BY SchemaName,TableName,c.ORDINAL_POSITION
```

## 查詢所有外鍵

```sql
SELECT  
     KCU1.CONSTRAINT_NAME AS FK_CONSTRAINT_NAME 
    ,KCU1.TABLE_NAME AS FK_TABLE_NAME 
    ,KCU1.COLUMN_NAME AS FK_COLUMN_NAME 
    ,KCU1.ORDINAL_POSITION AS FK_ORDINAL_POSITION 
    ,KCU2.CONSTRAINT_NAME AS REFERENCED_CONSTRAINT_NAME 
    ,KCU2.TABLE_NAME AS REFERENCED_TABLE_NAME 
    ,KCU2.COLUMN_NAME AS REFERENCED_COLUMN_NAME 
    ,KCU2.ORDINAL_POSITION AS REFERENCED_ORDINAL_POSITION 
FROM INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS AS RC 

INNER JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS KCU1 
    ON KCU1.CONSTRAINT_CATALOG = RC.CONSTRAINT_CATALOG  
    AND KCU1.CONSTRAINT_SCHEMA = RC.CONSTRAINT_SCHEMA 
    AND KCU1.CONSTRAINT_NAME = RC.CONSTRAINT_NAME 

INNER JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS KCU2 
    ON KCU2.CONSTRAINT_CATALOG = RC.UNIQUE_CONSTRAINT_CATALOG  
    AND KCU2.CONSTRAINT_SCHEMA = RC.UNIQUE_CONSTRAINT_SCHEMA 
    AND KCU2.CONSTRAINT_NAME = RC.UNIQUE_CONSTRAINT_NAME 
    AND KCU2.ORDINAL_POSITION = KCU1.ORDINAL_POSITION 
ORDER BY FK_TABLE_NAME,FK_COLUMN_NAME
```

## 查詢PK或FK設定錯誤

假定為Id結尾的欄位若且唯若是PK或FK

```sql
SELECT * FROM(
SELECT SchemaName = c.table_schema,
TableName = c.table_name,
ColumnIndex = c.ORDINAL_POSITION,
ColumnName = c.column_name,
DataType = data_type + (CASE WHEN c.CHARACTER_MAXIMUM_LENGTH IS NOT NULL THEN
(CASE WHEN c.CHARACTER_MAXIMUM_LENGTH = -1 THEN
'(max)'
ELSE
'(' + CONVERT(varchar(10), c.CHARACTER_MAXIMUM_LENGTH) + ')'
END)
ELSE
''
END),
NullAble = CASE WHEN c.IS_NULLABLE = 'YES' THEN 'NULL' ELSE 'NOT NULL' END,
IsIdentity = COLUMNPROPERTY(object_id(c.TABLE_SCHEMA+'.'+c.TABLE_NAME), c.COLUMN_NAME, 'IsIdentity'),
PK = CASE WHEN pk.COLUMN_NAME IS NOT NULL THEN 'PK ' + CONVERT(varchar(10), pk.ORDINAL_POSITION) ELSE '' END,
FK = CASE WHEN fk.COLUMN_NAME IS NOT NULL THEN 'FK' ELSE '' END
FROM information_schema.columns c
INNER JOIN information_schema.tables t
ON c.table_name = t.table_name
AND c.table_schema = t.table_schema
AND t.table_type = 'BASE TABLE'
LEFT JOIN (
SELECT
ku.TABLE_CATALOG,
ku.TABLE_SCHEMA,
ku.TABLE_NAME,
ku.COLUMN_NAME,
ku.ORDINAL_POSITION
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS AS tc
INNER JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS ku
ON tc.CONSTRAINT_TYPE = 'PRIMARY KEY'
AND tc.CONSTRAINT_NAME = ku.CONSTRAINT_NAME
) pk
ON c.TABLE_CATALOG = pk.TABLE_CATALOG
AND c.TABLE_SCHEMA = pk.TABLE_SCHEMA
AND c.TABLE_NAME = pk.TABLE_NAME
AND c.COLUMN_NAME = pk.COLUMN_NAME
LEFT JOIN(
SELECT DISTINCT
KCU1.TABLE_CATALOG,
KCU1.TABLE_SCHEMA,
KCU1.TABLE_NAME,
KCU1.COLUMN_NAME
FROM INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS AS RC
INNER JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS KCU1
ON KCU1.CONSTRAINT_CATALOG = RC.CONSTRAINT_CATALOG
AND KCU1.CONSTRAINT_SCHEMA = RC.CONSTRAINT_SCHEMA
AND KCU1.CONSTRAINT_NAME = RC.CONSTRAINT_NAME
) fk
ON c.TABLE_CATALOG = fk.TABLE_CATALOG
AND c.TABLE_SCHEMA = fk.TABLE_SCHEMA
AND c.TABLE_NAME = fk.TABLE_NAME
AND c.COLUMN_NAME = fk.COLUMN_NAME
) a
WHERE (ColumnName LIKE '%Id' AND PK = '' AND FK = '') OR
(ColumnName NOT LIKE '%Id' AND (PK != '' OR FK != ''))
ORDER BY TableName, ColumnIndex
```

## 檢查叢集索引用sn是否有自動增長

```sql
SELECT * FROM(
SELECT SchemaName = c.table_schema,
TableName = c.table_name,
ColumnIndex = c.ORDINAL_POSITION,
ColumnName = c.column_name,
DataType = data_type + (CASE WHEN c.CHARACTER_MAXIMUM_LENGTH IS NOT NULL THEN
(CASE WHEN c.CHARACTER_MAXIMUM_LENGTH = -1 THEN
'(max)'
ELSE
'(' + CONVERT(varchar(10), c.CHARACTER_MAXIMUM_LENGTH) + ')'
END)
ELSE
''
END),
NullAble = CASE WHEN c.IS_NULLABLE = 'YES' THEN 'NULL' ELSE 'NOT NULL' END,
IsIdentity = COLUMNPROPERTY(object_id(c.TABLE_SCHEMA+'.'+c.TABLE_NAME), c.COLUMN_NAME, 'IsIdentity'),
PK = CASE WHEN pk.COLUMN_NAME IS NOT NULL THEN 'PK ' + CONVERT(varchar(10), pk.ORDINAL_POSITION) ELSE '' END,
FK = CASE WHEN fk.COLUMN_NAME IS NOT NULL THEN 'FK' ELSE '' END
FROM information_schema.columns c
INNER JOIN information_schema.tables t
ON c.table_name = t.table_name
AND c.table_schema = t.table_schema
AND t.table_type = 'BASE TABLE'
LEFT JOIN (
SELECT
ku.TABLE_CATALOG,
ku.TABLE_SCHEMA,
ku.TABLE_NAME,
ku.COLUMN_NAME,
ku.ORDINAL_POSITION
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS AS tc
INNER JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS ku
ON tc.CONSTRAINT_TYPE = 'PRIMARY KEY'
AND tc.CONSTRAINT_NAME = ku.CONSTRAINT_NAME
) pk
ON c.TABLE_CATALOG = pk.TABLE_CATALOG
AND c.TABLE_SCHEMA = pk.TABLE_SCHEMA
AND c.TABLE_NAME = pk.TABLE_NAME
AND c.COLUMN_NAME = pk.COLUMN_NAME
LEFT JOIN(
SELECT DISTINCT
KCU1.TABLE_CATALOG,
KCU1.TABLE_SCHEMA,
KCU1.TABLE_NAME,
KCU1.COLUMN_NAME
FROM INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS AS RC
INNER JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS KCU1
ON KCU1.CONSTRAINT_CATALOG = RC.CONSTRAINT_CATALOG
AND KCU1.CONSTRAINT_SCHEMA = RC.CONSTRAINT_SCHEMA
AND KCU1.CONSTRAINT_NAME = RC.CONSTRAINT_NAME
) fk
ON c.TABLE_CATALOG = fk.TABLE_CATALOG
AND c.TABLE_SCHEMA = fk.TABLE_SCHEMA
AND c.TABLE_NAME = fk.TABLE_NAME
AND c.COLUMN_NAME = fk.COLUMN_NAME
) a
WHERE IsIdentity = 0 AND ColumnName = 'sn'
ORDER BY TableName, ColumnIndex
```
## CTE遞迴
### 列出樹狀節點表的節點下所有節點(由上而下)
``` sql
WITH TEMP AS (
	SELECT
            Id,
            Name,
            Pid,
            Id AS [Index]
	FROM [Node]
	UNION ALL
	SELECT
            [Node].Id,
            [Node].Name,
            [Node].Pid,
            [TEMP].[Index]
	FROM [Node]
	INNER JOIN TEMP ON [Node].Pid = [TEMP].Id
)
SELECT * FROM TEMP
```
#### 原表
| Id | Name  | Pid  |
|----|-------|------|
| 1  | A     | NULL |
| 2  | A-A   | 1    |
| 3  | A-B   | 1    |
| 4  | A-A-A | 2    |
| 5  | A-A-B | 2    |
| 6  | A-B-A | 3    |
| 7  | A-B-B | 3    |
| 8  | B     | NULL |
| 9  | B-A   | 8    |
| 10 | B-B   | 8    |

#### 結果

| Id | Name  | Pid  | Index |
|----|-------|------|-------|
| 1  | A     | NULL | 1     |
| 2  | A-A   | 1    | 2     |
| 3  | A-B   | 1    | 3     |
| 4  | A-A-A | 2    | 4     |
| 5  | A-A-B | 2    | 5     |
| 6  | A-B-A | 3    | 6     |
| 7  | A-B-B | 3    | 7     |
| 8  | B     | NULL | 8     |
| 9  | B-A   | 8    | 9     |
| 10 | B-B   | 8    | 10    |
| 9  | B-A   | 8    | 8     |
| 10 | B-B   | 8    | 8     |
| 6  | A-B-A | 3    | 3     |
| 7  | A-B-B | 3    | 3     |
| 4  | A-A-A | 2    | 2     |
| 5  | A-A-B | 2    | 2     |
| 2  | A-A   | 1    | 1     |
| 3  | A-B   | 1    | 1     |
| 6  | A-B-A | 3    | 1     |
| 7  | A-B-B | 3    | 1     |
| 4  | A-A-A | 2    | 1     |
| 5  | A-A-B | 2    | 1     |
### 列出樹狀節點表的節點上所有節點(由下而上)
``` sql
WITH TEMP AS (
	SELECT
            Id,
            Name,
            Pid,
            Id AS [Index]
	FROM [Node]
	UNION ALL
	SELECT
            [Node].Id,
            [Node].Name,
            [Node].Pid,
            [TEMP].[Index]
	FROM [Node]
	INNER JOIN TEMP ON [Node].Id = [TEMP].Pid
)
SELECT * FROM TEMP
```