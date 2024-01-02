# Windows 우분투/ES 설치

참고 : 2023.08월 기준 Springboot에 연동 하기 위해서는 elasticsearch-8-7-1을 설치해야 합니다

## 1. 다운로드

다운 로드 : [Elasticsearch 8.7.1](https://www.elastic.co/kr/downloads/past-releases/elasticsearch-8-7-1)

Linux [다운로드](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.7.1-linux-x86\_64.tar.gz)&#x20;

## 2. 자바 설치&#x20;

apt-get update\
apt-get upgrade\
apt install openjdk-17-jdk openjdk-17-jre

## 3. Elactic ELK 설치

참고사이트 : [https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html)



### 3.1  Elasticsearch &#x20;

#### 3-1-1.  Elasticsearch 다운 로드&#x20;

```bash
 - wget 
https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.9.0-linux-x86_64.tar.gz

2. Elasticsearch 체크섬 다운로드
 - wget 
https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.9.0-linux-x86_64.tar.gz.sha512

3. 체크 
 - shasum -a 512 -c elasticsearch-8.9.0-linux-x86_64.tar.gz.sha512 
4. 설치 
 -- tar -xzf elasticsearch-8.9.0-linux-x86_64.tar.gz
5. 실행 
 - cd elasticsearch-8.9.0/ 
 - ./bin/elasticsearch
```

#### 3-1-2. 가상 메모리 부족 나오면 : &#x20;

```bash
sudo sysctl -w vm.max_map_count=262144
또는 
$ sudo vim /etc/sysctl.conf 실행 후 아래 내용 추가 혹은 수정
vm.max_map_count=262144
```

#### &#x20;3-1-3. 설치 후 아래 메세지 메모모

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\
✅ Elasticsearch security features have been automatically configured!\
✅ Authentication is enabled and cluster connections are encrypted.\
\
ℹ️ Password for the elastic user (reset with \`bin/elasticsearch-reset-password -u elastic\`):\
ji\_DrcMq5xL+aCs+6VjO\
\
ℹ️ HTTP CA certificate SHA-256 fingerprint:\
8ff2a69c78554e2fbc80baa50742fe10de86583b13df24efd4c7f191f804fada\
\
ℹ️ Configure Kibana to use this cluster:\
• Run Kibana and click the configuration link in the terminal when Kibana starts.\
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30\
minutes):\
eyJ2ZXIiOiI4LjkuMCIsImFkciI6WyIxNzIuMTkuMTY0LjEzMjo5MjAwIl0sImZnciI6IjhmZjJhNjljNzg1NTRlMmZiYzgwYmFh\
NTA3NDJmZTEwZGU4NjU4M2IxM2RmMjRlZmQ0YzdmMTkxZjgwNGZhZGEiLCJrZXkiOiJEeGl3LVlrQkF0OTFSR2xPanRseTowaHg4ek\
wybVJHVy1JeVpKaXpFZzBBIn0=\
\
ℹ️ Configure other nodes to join this cluster:\
• Copy the following enrollment token and start new Elasticsearch nodes with \`bin/elasticsearch --enro\
llment-token \<token>\` (valid for the next 30 minutes):\
eyJ2ZXIiOiI4LjkuMCIsImFkciI6WyIxNzIuMTkuMTY0LjEzMjo5MjAwIl0sImZnciI6IjhmZjJhNjljNzg1NTRlMmZiYzgwYmFh\
NTA3NDJmZTEwZGU4NjU4M2IxM2RmMjRlZmQ0YzdmMTkxZjgwNGZhZGEiLCJrZXkiOiJEaGl3LVlrQkF0OTFSR2xPanRseTowbzlKNk\
ZHRFJJNnljdlhxXzdDZVZ3In0=\
\
If you're running in Docker, copy the enrollment token and run:\
\`docker run -e "ENROLLMENT\_TOKEN=\<token>" docker.elastic.co/elasticsearch/elasticsearch:8.9.0\`\
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

&#x20;

\*\* kibana \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\
참고사이트 : [https://www.elastic.co/guide/en/kibana/current/targz.html](https://www.elastic.co/guide/en/kibana/current/targz.html)

### 3.2 Kabana 설치

```bash
curl -O 
https://artifacts.elastic.co/downloads/kibana/kibana-8.9.0-linux-x86_64.tar.gz

2. 체크섬 
 - curl 
https://artifacts.elastic.co/downloads/kibana/kibana-8.9.0-linux-x86_64.tar.gz.sha512
 | shasum -a 512 -c - 
3. 설치;
 - tar -xzf kibana-8.9.0-linux-x86_64.tar.gz 
4. 실행 
 - cd kibana-8.9.0/ 
```
