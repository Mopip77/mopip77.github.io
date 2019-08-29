---
layout: post
title: mysql8由于更换密码算法NaviCat连不上
tags:
- mysql
---

1. 进入mysql
2. 设置权限(此处为root用户)
```mysql
mysql> grant all PRIVILEGES on *.* to root@'%' WITH GRANT OPTION;
Query OK, 0 rows affected (0.01 sec)
```

3. 更新密码算法
```mysql
mysql> grant all PRIVILEGES on *.* to root@'%' WITH GRANT OPTION;
Query OK, 0 rows affected (0.01 sec)
 
mysql> ALTER user 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
Query OK, 0 rows affected (0.11 sec)
 
mysql> ALTER user 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.11 sec)
 
mysql>  FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)
```