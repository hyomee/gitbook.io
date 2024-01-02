# Redis 설치

## 1. Redis 설치

<pre class="language-null"><code class="lang-null"><strong>$ sudo apt-get install redis-server
</strong></code></pre>

## 3. Redis 외부 접근 허용

```bash
$ sudo nano /etc/redis/redis.conf

bind 0.0.0.0 ::1
...(생략)
requirepass [비밀번호 지정]
...(생략) 
port [포트지정]
```

## 4. Redis 시작

```null
$ sudo service redis-server restart
```

## 5. Redis 시스템 등록

```
$ sudo systemctl status redis-server
● redis-server.service - Advanced key-value store
     Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2023-09-14 17:34:28 KST; 2min 25s ago
       Docs: http://redis.io/documentation,
             man:redis-server(1)
   Main PID: 900 (redis-server)
     Status: "Ready to accept connections"
      Tasks: 5 (limit: 9297)
     Memory: 9.6M
     CGroup: /system.slice/redis-server.service
             └─900 "/usr/bin/redis-server 0.0.0.1:6379" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >

```

## 6. Redis 시스템 6379 포트 오픈

```
sudo ufw allow proto tcp from 192.168.121.0/24 to any port 6379
```

## 7. redis-cli

AUTH : 비밀번호

KEYS \*

