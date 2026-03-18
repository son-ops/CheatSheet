## Schema Enumeration Cheatsheet
* [Dump database name](#1-lấy-database--schema-hiện-tại)
* [Dump table name](#3-lấy-table-theo-database--schema)
* [Dump column name](#4-lấy-column--cấu-trúc-của-table-cụ-thể)

### 1. Lấy database / schema hiện tại

| DBMS | Query |
|---|---|
| MySQL | `SELECT DATABASE();` |
| PostgreSQL | `SELECT current_database();` / `SELECT current_schema();` |
| MSSQL | `SELECT DB_NAME();` |
| Oracle | `SELECT USER FROM dual;` |
| SQLite | Không áp dụng theo kiểu server DBMS truyền thống |

---

### 2. Lấy tất cả database / schema

| DBMS | Query |
|---|---|
| MySQL | `SELECT database_name FROM mysql.innodb_table_stats;` |
| MySQL | `SELECT schema_name FROM information_schema.schemata;` |
| PostgreSQL | `SELECT datname FROM pg_database;` / `SELECT schema_name FROM information_schema.schemata;` |
| MSSQL | `SELECT name FROM sys.databases;` / `SELECT name FROM sys.schemas;` |
| Oracle | `SELECT username FROM all_users;` |
| SQLite | Không áp dụng |

---

### 3. Lấy table theo database / schema

| DBMS | Query |
|---|---|
| MySQL | `SELECT table_name FROM mysql.innodb_table_stats WHERE database_name = '<database_name>';` |
| MySQL | `SELECT table_name FROM information_schema.tables WHERE table_schema = '<database_name>';` |
| PostgreSQL | `SELECT table_name FROM information_schema.tables WHERE table_schema = '<schema_name>';` |
| MSSQL | `SELECT table_name FROM information_schema.tables WHERE table_schema = '<schema_name>';` / `SELECT table_schema, table_name FROM information_schema.tables WHERE table_type = 'BASE TABLE';` |
| Oracle | `SELECT table_name FROM all_tables WHERE owner = '<schema_name>';` / `SELECT owner, table_name FROM all_tables;` |
| SQLite | `SELECT tbl_name FROM sqlite_master WHERE type='table';` / `SELECT group_concat(tbl_name) FROM sqlite_master WHERE type='table' AND tbl_name NOT LIKE 'sqlite_%';` |

---

### 4. Lấy column / cấu trúc của table cụ thể

| DBMS | Query |
|---|---|
| MySQL | `SELECT column_name, data_type FROM information_schema.columns WHERE table_schema = '<database_name>' AND table_name = '<table_name>';` |
| PostgreSQL | `SELECT column_name, data_type FROM information_schema.columns WHERE table_schema = '<schema_name>' AND table_name = '<table_name>';` |
| MSSQL | `SELECT column_name, data_type FROM information_schema.columns WHERE table_schema = '<schema_name>' AND table_name = '<table_name>';` |
| Oracle | `SELECT owner, column_name, data_type FROM all_tab_columns WHERE owner = '<SCHEMA_NAME>' AND table_name = '<TABLE_NAME>';` |
| SQLite | `SELECT sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name='<table_name>';` / `SELECT name FROM PRAGMA_TABLE_INFO('<table_name>');`|