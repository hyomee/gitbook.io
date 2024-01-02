# Springboot의 사용자 정의 파일에 등록

resource 폴더 밑에 사용자 정의 파일을 생성 하고 다음과 같은 속성을 정의 합니다. ( 예제 : /config/custom/el.yml )

```yaml
es:	
  # ElasticSearch에 할당 당 ip (or 도메인 주소 )								# 사용자 정의 root 
  host: 192.168.210.52	 
  
  # ElasticSearch에 할당 port
  port: 9200	 
  
  # ElasticSearch에 접근 할 사용자 아이디
  username: elastic	 
  
  # ElasticSearch에 접근 할 사용자 비밀번호
  password: =dq2XKtp_p2DF_2QiLys  
  
  # ElasticSearch에 접근 할 스키마
  schemeName: https	
  
  # pwd, fingerprint, ca ( 인증 방법 )  
  ssl: pwd  	 
  
  fingerprint: 190f95778c4f9b2d5247d940b9665c3494b6f930914796cc4274c432150440a7
  cadrt: C:\PRJ\xxxx\elk\http_ca.crt
```

사용자 아이디, 비밀번호, fingerprint와 ca 파일은 Elasticsearch 최초 설치시 나오는 메세지로 해당 부분에 있는 값을 사용자 정의 파일에 기재 합니다.



참고 : [https://www.elastic.co/guide/index.html](https://www.elastic.co/guide/index.html)
