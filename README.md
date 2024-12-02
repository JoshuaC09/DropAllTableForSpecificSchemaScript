-- First, drop all foreign key constraints
DECLARE @sql NVARCHAR(MAX) = '';

SELECT @sql += 'ALTER TABLE ' + QUOTENAME(OBJECT_SCHEMA_NAME(parent_object_id)) + '.' + QUOTENAME(OBJECT_NAME(parent_object_id)) + 
    ' DROP CONSTRAINT ' + QUOTENAME(name) + ';'
FROM sys.foreign_keys
WHERE referenced_object_id IN (
    SELECT object_id 
    FROM sys.tables t
    JOIN sys.schemas s ON t.schema_id = s.schema_id
    WHERE s.name = 'core'
);

EXEC sp_executesql @sql;

-- Then drop all tables with core schema
SET @sql = '';

SELECT @sql += 'DROP TABLE IF EXISTS ' + QUOTENAME(s.name) + '.' + QUOTENAME(t.name) + ';' + CHAR(13)
FROM sys.tables t
JOIN sys.schemas s ON t.schema_id = s.schema_id
WHERE s.name = 'core';

EXEC sp_executesql @sql;
