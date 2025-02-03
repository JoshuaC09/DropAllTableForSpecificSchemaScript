-- Drop sequence first
DECLARE @dropSequence NVARCHAR(MAX) = '';
SELECT @dropSequence = 'DROP SEQUENCE IF EXISTS core.EmployeeCodeSequence;';
EXEC sp_executesql @dropSequence;

-- Then your existing code to drop FKs and tables
DECLARE @sql NVARCHAR(MAX) = '';
SELECT @sql += 'ALTER TABLE ' + QUOTENAME(OBJECT_SCHEMA_NAME(parent_object_id)) + '.' + QUOTENAME(OBJECT_NAME(parent_object_id)) + ' DROP CONSTRAINT ' + QUOTENAME(name) + ';' 
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
