# 설치

**RabbitMQ** 서버를 설치하는 가이드를 바탕으로 설치 진행을 합니다.

참고: [Debian 및 Ubuntu에 설치](https://www.rabbitmq.com/docs/install-debian#package-dependencies)

## 1. 필수 종속 설치

```sh
sudo apt-get update -y

sudo apt-get install curl gnupg -y
```

## 2. Enable apt HTTPS Transport

apt가 Cloudsmith.io 미러 또는 런치 패드에서 RabbitMQ 및 Erlang 패키지를 다운로드 할 수 있도록하려면 패키지를 설치해야 합니다

```sh
sudo apt-get install apt-transport-https
```

## 3. 리포지토리 서명 키 추가

모든 제 3 자 apt 저장소와 마찬가지로 RabbitMQ 및 Erlang 패키지 저장소를 설명하는 파일 디렉터리 아래에 배치해야 하는데   권장 위치 /etc/apt/sources.list.d/, /etc/apt/sources.list.d/rabbitmq.list 입니다.

```sh
## Team RabbitMQ's main signing key
curl -1sLf "https://keys.openpgp.org/vks/v1/by-fingerprint/0A9AF2115F4687BD29803A206B73A36E6026DFCA" 
sudo gpg --dearmor 
sudo tee /usr/share/keyrings/com.rabbitmq.team.gpg > /dev/null

## Community mirror of Cloudsmith: modern Erlang repository
curl -1sLf https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key
sudo gpg --dearmor 
sudo tee /usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg > /dev/null

## Community mirror of Cloudsmith: RabbitMQ repository
curl -1sLf https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key 
sudo gpg --dearmor 
sudo tee /usr/share/keyrings/rabbitmq.9F4587F226208342.gpg > /dev/null
```

apt 리포지토리를 소스 목록 디렉터리(아래 )에 추가하려면 다음을 사용합니다 (/etc/apt/sources.list.d)

| Release   | Distribution |
| --------- | ------------ |
| 우분투 23.04 | `jammy`      |
| 우분투 22.04 | `jammy`      |
| 우분투 20.04 | `focal`      |
| 우분투 18.04 | `bionic`     |
| 데비안 책벌레   | `bullseye`   |
| 데비안 불스아이  | `bullseye`   |
| 데비안 시드    | `bullseye`   |

$distribution 에 부분에 서버 버전에 따른 Distribution를 변경 해야 합니다.

```sh
sudo tee /etc/apt/sources.list.d/rabbitmq.list <<EOF
## Provides modern Erlang/OTP releases from a Cloudsmith mirror
##
deb [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu $distribution main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu $distribution main

# another mirror for redundancy
deb [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu $distribution main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu $distribution main

## Provides RabbitMQ from a Cloudsmith mirror
##
deb [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu $distribution main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu $distribution main

# another mirror for redundancy
deb [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu $distribution main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu $distribution main
EOF
```

## 4. Erlang Packages, RabbitMQ Server 설치하기 <a href="#erlang-packages-rabbitmq-server" id="erlang-packages-rabbitmq-server"></a>

```sh
sudo apt-get update
sudo apt-get install rabbitmq-server
```

## 5. RabbitMQ Server 시작 <a href="#erlang-packages-rabbitmq-server" id="erlang-packages-rabbitmq-server"></a>

```sh
service rabbitmq-server start
```

## 6. Management UI 플러그인 활성화

RabbitMQ는 Management 라는 UI 관리 도구를 사용하기 위해서는 [RabbitMQ Plugin](https://www.rabbitmq.com/docs/management)을 활성화 시켜야 한다.

```sh
rabbitmq-plugins enable rabbitmq_management
```

## 7. Management UI 사용자 추가

Tag 종류는 공식 가이드( [Management Plugin - Access and Permissions](https://www.rabbitmq.com/management.html#permissions) )에서 더 확인 할 수 있고, 아래에서는 관리자(‘administrator’) 권한을 사용 할 수 있다.

```sh
# 사용자 목록
$ sudo rabbitmqctl list_users
    
# 사용자 추가
$ sudo rabbitmqctl add_user hong 'hongrabbit'
    
# 사용자 태그 추가
$ sudo rabbitmqctl set_user_tags hong administrator
```

## 8. 포트 설정

방화벽을 사용하고 있다면 아래 포트를 열어줘야 한다.

* 4369: epmd , 여러 rabbitmq 서버끼리 서로를 찾을 수 있는 네임 서버 역할을 하는 데몬에서 사용
* 5672, 5671: AMQP 를 사용한 메시지 전달
* 25672: inter-node 와 CLI Tool 연결
* 15672: HTTP API, Management UI

## 9. Virtual Hosts

connection, exchange, queue, binding, user, policy 들을 **virtual hosts** 를 통해 논리적인 그룹으로 분리해서 운영할 수 있습니다. 설정은 [이곳](https://www.rabbitmq.com/docs/vhosts)에서 확인 할 수 있습니다,

## 10. 로그&#x20;

* 위치: /var/log/rabbitmq
* 로그 파일 로테이션을 위해 `logrotate` 를 사용해서 변경 할 수 있습니다.
* 설정: `/etc/logrotate.d/rabbitmq-server` 에서 확인하고 변경할 수 있습니다.

```
/var/log/rabbitmq/*.log {
        daily
        missingok
        compress
        delaycompress
        notifempty
}
```

## 11. 관리

* rabbitmq 상태:  sudo rabbitmq-diagnostics status
* rabbitmq  환경 구성: [https://www.rabbitmq.com/docs/configure](https://www.rabbitmq.com/docs/configure)

참고: [rabbitmqctl](https://www.rabbitmq.com/rabbitmqctl.8.html)
