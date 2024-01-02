---
description: 윈도우에 WSL, 우분투 기반 MariaDB 설치
---

# MariaDB 설치

윈도우에 MariaDB를 설치하기 방법은 다음과 같습니다.

* 윈도우 버젼의 MariaDB를 설치
* &#x20;WSL설치, 우분투 설치 후 MariaDB를 설치

본 문서에서는  WSL, 우분투가 설치있는 윈도우에 설치하는 과정을 설명 합니다.

## 1.      우분투 저장소에 있는 MariaDB를 설치 &#x20;

<pre class="language-bash"><code class="lang-bash">// 1. apt update
<strong>$ sudo apt update
</strong>
// 2. MariaDB 설치
$ sudo apt install mariadb-server -Y

// 3. MariaDB 시작
$ sudo systemctl start mariadb

// 4. MariaDB 시스템에 등록 - 부팅 시 자동 시작 설정
$ sudo systemctl enable mariadb 

Synchronizing state of mariadb.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable mariadb
</code></pre>

## 2. MariaDB 설치 확인

### 2-1. 네트워크 확인

```
$ netstat -anp | grep 3306
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -
```

### 2-2. 권한 없이 접근 하는 방법 :&#x20;

```bash
$ sudo mariadb
Welcome to the MariaDB monitor. Commands end with ; or \g. 
Your MariaDB connection id is 31 Server version: 10.6.12-MariaDB-0ubuntu0.22.04.1 Ubuntu 22.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

### 2-1. 데이터 베이스 조회

```bash
MariaDB [(none)]> show databases; 
+--------------------+ 
| Database           | 
+--------------------+ 
| information_schema | 
| mysql              | 
| performance_schema | 
| sys                | 
+--------------------+ 
4 rows in set (0.002 sec)
```

### 2-2. 사용자 정보 조회

```
MariaDB [(none)]> select  user , host, plugin , authentication_string from mysql.user;
+-------------+-----------+-----------------------+-----------------------+
| User        | Host      | plugin                | authentication_string |
+-------------+-----------+-----------------------+-----------------------+
| mariadb.sys | localhost | mysql_native_password |                       |
| root        | localhost | mysql_native_password | invalid               |
| mysql       | localhost | mysql_native_password | invalid               |
+-------------+-----------+-----------------------+-----------------------+
3 rows in set (0.001 sec)
```

root 계정은 localhost host에서  접근이 되도록 되어 있고 비밀번호가 없습니다.&#x20;

plugin이 unix\_socket로 되어 있으면 mysql -u root -p로 접근  시&#x20;

```
$ mysql -u root -p
Enter password:
ERROR 1698 (28000): Access denied for user 'root'@'localhost'
```

오류가 납니다.&#x20;

### 2-3. 비밀번호 설정 및 Plugin 변경

<pre><code><strong>use mysql;
</strong>update user set plugin = 'mysql_native_password' where User='root';
<strong>set password for 'root'@'localhost' = password('sang0237!');
</strong><strong>FLUSH PRIVILEGES;
</strong></code></pre>

*   확인 방법 : 다음 명령으로 mysql에 접근 하고 비밀번호를 입력한다.\


    ```bash
    $ mysql -u root -p
    Enter password:
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 35
    Server version: 10.6.12-MariaDB-0ubuntu0.22.04.1 Ubuntu 22.04

    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    MariaDB [(none)]>
    ```

### 2-4. 사용자 생성

root 계정을 외부에서 접근하게 하는 것은 보안상 문제가 있습니다. 외부에서 접근 가능한 계정을 생성 하고 외부 접근을 허용해야 합니다.

```
MariaDB [mysql]> CREATE USER 'hyomee'@'%' IDENTIFIED BY 'password';
MariaDB [mysql]> set password for 'hyomee'@'%' = password('password'); 
Query OK, 0 rows affected (0.012 sec)
MariaDB [mysql]> flush privileges; 
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> CREATE USER 'hyomee'@'localhost' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.004 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> set password for 'hyomee'@'localhost' = password('sang0237!');
Query OK, 0 rows affected (0.004 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.001 sec)
```

### 2.5 db 생성 및 사용자 설정

```
MariaDB [(none)]> create database hyomeedb;
Query OK, 1 row affected (0.007 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> grant all privileges on hyomeedb.* to 'hyomee'@'%';
Query OK, 0 rows affected (0.005 sec)

MariaDB [(none)]> grant all privileges on hyomeedb.* to 'hyomee'@'localhost';
Query OK, 0 rows affected (0.005 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.001 sec)
```
