# MySQL 설치

## 1, 우분투 서버 업데이트 및 Mysql -server 설치&#x20;

```shell
$sudo apt-get update 
$sudo apt-get install mysql-serve
```

* cnf 위치: /etc/mysql/mysql.cnf &#x20;
* 설치위치: /usr/bin/mysql

### 2.Mysql 기본 세팅

*   외부 접속 기능 설정 (포트 3306 오픈):  &#x20;

    ```sh
    $sudo ufw allow mysql
    ```
*   시작 :&#x20;

    ```sh
    $sudo systemctl start mysql
    ```
*   Ubuntu 서버 재시작시 Mysql 자동 재시작 :&#x20;

    ```sh
    $sudo systemctl enable mysql
    ```

## 3. 접속

```sh
$sudo mysql -u root -p
```

## 4. DB 기본작업

*   버전 확인

    ```sh
    mysql> show variables like "%version%";
    ```

<figure><img src="../.gitbook/assets/image.png" alt="" width="412"><figcaption></figcaption></figure>

*   사용자 정보 확인

    ```sh
    mysql> SELECT User, Host, Plugin, authentication_string FROM mysql.user;
    ```

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

*   Mysql 비번 변경 방법

    ```sh
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '변경비밀번호';
    ```
*   권한 재 설정 후 재 실행을 합니다.

    ```sh
    $ mysql_secure_installation
    $ sudo service mysql restart
    ```
*   DB 생성

    <pre class="language-sh"><code class="lang-sh">mysql> CREATE DATABASE 테이블명
    <strong>mysql> FLUSH PRIVILEGES;
    </strong></code></pre>
*   사용자 권한 보는 법

    ```sh
    mysql> SHOW GRANTS FOR 'root'@'localhost';
    ```
*   사용자 등록 및 삭제&#x20;

    ```sh
    mysql> CREATE USER 'hong'@'localhost' IDENTIFIED WITH mysql_native_password BY '등록할비밀번호';
    mysql> FLUSH PRIVILEGES;

    ** 삭제 : drop user 'hong'@'localhost';
    ```
*   사용자에게 db 권한 설정

    ```sh
    mysql> GRANT ALL PRIVILEGES ON hongdb.* TO 'hong'@'localhost';
    mysql> FLUSH PRIVILEGES;
    mysql> SHOW GRANTS FOR 'hong'@'localhost';

    ```

