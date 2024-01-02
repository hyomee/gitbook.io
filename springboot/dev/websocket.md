# WebSocket

웹 브라우저에서 클라리언트와 서버간의 통신은 요청에 의한 응답을 주는 방법을 다음 절차를 따릅니다.

<figure><img src="https://blog.kakaocdn.net/dn/QdADJ/btsscFhP2EX/6kEBk4EjGQk4V0UkUZK2AK/img.png" alt=""><figcaption></figcaption></figure>

1. Http Client에서 Http Server로 데이터를 요청 합니다.
2. Http Server은 Http Client에서 전달 받은 전문을 해석 하여 결과로 정보를 돌려 줍니다.

XMLHttpRequest 객체를 마이크로소프트 에서 개발하여 사용하던 것을 World Wide Web Consortium이 2006년에 XMLHttpRequest 객체의 작업 사양을 발표한 이후 서버에서 데이터를 가져온후 페이지 전체를 로드 할 필요가 없어지게 되었다 이 시점을 기준으로 비동기 요청을 웹 브라우저에서 할 수 있게 되어 풍부한 웹 페이지가 탄생하게 됩니다.&#x20;

## **1. polling**

<figure><img src="https://blog.kakaocdn.net/dn/cek5zV/btssfFIDYh5/e9Gvah3kQp3OJK3x4WzZik/img.png" alt="" height="308" width="334"><figcaption></figcaption></figure>

클라이언트에서 서버로 전통적인 요청/응답 형태로 새 테이터를 받아서 표시 하는 방법으로 다음과 같은 문제가 있습니다.

* 요청 할 떄 마다 새로운 연결을 하여 많은 사용자가 접속 하면 서버에 부하를 증가 시킵니다.
* 즉. 연결에 따른 HTTP 헤더 구문 분석 , 새 데이터에 대한 쿼리 수행, 빈 데이터의 전달, 연결 해제에 따른 리소스 낭비가 발생 합니다.
* 자바 스크립트의 타이머를 이용한 개발 &#x20;

## **2. Long polling**

<figure><img src="https://blog.kakaocdn.net/dn/dkNBNS/btssfVErCTm/URZDsOUaSEU6et7OXK68Ek/img.png" alt="" height="272" width="344"><figcaption></figcaption></figure>

Client에서 서버로 API요청시 즉각적인 응답을 받는 대시 HTTP 연결을 유지하는 방식으로 데이터가 클라이언트로 요청에 의한 응답으로 보내 거나 시간 임계값에 도달하면 서버가 클라이언트로 응답할 수 있습니다.&#x20;

즉, 클라이언트는 최신 정보를 얻기 위해 서버에 하나의 요청만 보내면 되고 데이터를 받은 후 클라이언트는 새로운 요청을 시작하고 필요한 만큼 이 프로세스를 반복할 수 있습니다

* keep-Alive 응답을 수신할 때 Keep-Alive 헤더 사용
* 클라이언트의 요청 상태를 유지하고 /poll일정 시간이 지난 후 클라이언트를 엔드포인트로 계속 리디렉션합니다

## **3. Websocket**

<figure><img src="https://blog.kakaocdn.net/dn/dglg1h/btssgl3XlIf/4RFOA8H62PyzWphoR9SCvK/img.png" alt="" height="274" width="248"><figcaption></figcaption></figure>

TCP/IP 스택 위에 구축된 기술로 HTTP 프로토콜과의 유일한 관계는 HTTP 서버가 핸드셰이크를 해석하여 연결을 설정한다는 것입니다. 즉, 클라이언트와 서버 간의 연결은 어느 쪽이든 종료하기로 결정할 때까지 지속됩니다. {상태 저장형 양방향 전이중 프로토콜)[.](https://en.wikipedia.org/wiki/Duplex\_\(telecommunications\))&#x20;

반이중 솔루션인 롱 폴링과 달리 서버로부터 최신 정보를 받은 후 프로세스를 반복할 필요가 없습니다. WebSocket 기술을 사용하면 새 정보가 반환된 후에도 연결을 유지하고 양방향 업데이트를 수행할 수 있습니다. 클라이언트는 정보를 다시 서버로 보내고 동일한 요청에서 추가 정보를 수신할 수 있습니다.&#x20;

&#x20;

### 3-1. SpringBoot 를 사용한 예제

#### 3.1.1. 의존성

```gradle
implementation 'org.springframework.boot:spring-boot-starter-web:3.1.0'
implementation 'org.springframework.boot:spring-boot-starter-websocket:3.1.0'
```

#### 3.1.2. Websocket Configuration

&#x20;Soket 연결, SUBSCRIBE 연결 설정, PUBLISH 설정을 합니다.

```java
/**
 * 웹 소켓 구성 한다. Configure Spring for STOMP messaging
 * WebSocket 기능을 사용하도록 설정 : @EnableWebSocketMessageBroker
 */
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

  /**
   * WebSocketMessageBrokerConfigurer 구현
   * @param config
   */
  public void configureMessageBroker(MessageBrokerRegistry config) {
    // 메세지를 받을 경로로 스프링에서 제공 해주는 내장 브로커 사용
    //  “/queue”, “/topic”을 통해 1:1, 1:N 설정
    config.enableSimpleBroker("/topic");

    // 메세지를 보낼 때, 관련 경로를 설정
    // 클라이언트가 메세지를 보낼 떄, 경로 앞에 “/app”이 붙어있으면 Broker로 보내진다
    config.setApplicationDestinationPrefixes("/app");
  }


  /**
   * 소겟 연결과 관련된 설정
   * addEndpoint : 소켓 연결 uri
   * 연결 상태 : CONNECT : 연결 요청, CONNECTED : 연결 성공, ERROR : 연결 실패
   * 주의 사항
   *  - setAllowedOriginPatterns(”*”) : 소켓 또한 CORS 설정
   *  - withSockJS() : 소켓을 지원하지 않는 브라우저라면, sockJS를 사용하도록 설정
   * @param registry
   */
  public void registerStompEndpoints(StompEndpointRegistry registry) {
    registry.addEndpoint("/gs-websocket");
  }

}
```

메세지 받을 경로 : topic\
메세지 보내는 경로 ; /app\
연결 : gs-websocket

#### 3.1.3. Controller 작성

* &#x20;@MessageMapping : WebSocket을 사용할 때,  클라이언트로부터 메시지를 받았을 때 어떤 메서드를 실행할지 결정하는 역할
* @SendTo : 메시지를 보낼 대상을 지정. 대상은 토픽(topic)이나 큐(queue)의 형태로 지정할 수 있습니다.

```java
@RestController
@Slf4j
@RequiredArgsConstructor
public class controller {


  private final SimpMessagingTemplate template;

  // @MessageMapping : WebSocket을 사용할 때, 
  //        클라이언트로부터 메시지를 받았을 때 어떤 메서드를 실행할지 결정하는 역할
  // @SendTo : 메시지를 보낼 대상을 지정. 
  //        대상은 토픽(topic)이나 큐(queue)의 형태로 지정할 수 있습니다.
  @MessageMapping("/scheduledmsg")
  @SendTo("/topic/message")
  public GreetingMessag connect(HelloMessage message) throws Exception {
    Thread.sleep(1000); // simulated delay
    return GreetingMessag.builder()
            .content("Hello, " + HtmlUtils.htmlEscape(message.getName()) + "!").build();

  }


  @Scheduled(cron = "0/15 * * * * *" )
  public void sendMsg() {
    GreetingMessag greetingMessag = GreetingMessag.builder()
            .content("[메세지]" + UuidUtils.getUUID() + " ..... !").build();

    log.debug("cron 15초 이후 실행.. "
            + Thread.currentThread().getName() + " : "
            + LocalDateTime.now().format(DateTimeFormatter.ofPattern("YYYY-MM-dd'T'HH:mm:ss")));

    template.convertAndSend("/topic/message", greetingMessag);
  }
}
```

\*\* @MessageMapping 어노테이션을 사용할 때 주의해야 사항&#x20;

* 스프링 프레임워크에서 제공하는 어노테이션 중 하나로  WebSocket을 사용할 때, 클라이언트로부터 메시지를 받았을 때 어떤 메서드를 실행할지 결정
* 반드시 @Controller 적용된 클래스 내에 있어야 합니다.
* @MessageMapping  사용하는 메서드는 반드시 @SendTo 또는 SimpMessagingTemplate.convertAndSend() 메서드를 사용하여 클라이언트로 응답을 보내야 합니다.
* @MessageMapping 어노테이션을 사용하는 메서드의 파라미터는 다음과 같은 타입만 사용할 수 있습니다:
  * Message\<?>
  * String
  * byte\[]
  * InputStream
  * Reader
  * HttpHeaders
  * SimpMessageHeaderAccessor
  * @Payload로 표시된 객체
  * @Header로 표시된 객체
* @MessageMapping 예외가 발생하면 클라이언트로 에러 응답을 보내야 합니다

### **3-2. JavaScript 코드**&#x20;

```javascript
<script src="https://cdn.jsdelivr.net/npm/@stomp/stompjs@7.0.0/bundles/stomp.umd.min.js"></script>
  

// StompJs.Client url 선언
const stompClient = new StompJs.Client({
    brokerURL: 'ws://localhost:8080/gs-websocket'
});

// StompJs.Client 연결
stompClient.onConnect = (frame) => {
    setConnected(true);
    console.log('Connected: ' + frame);
    stompClient.subscribe('/topic/message', (message) => {
        showGreeting(JSON.parse(message.body).content);
    });
};

// WebSocket 에러 
stompClient.onWebSocketError = (error) => {
    console.error('Error with websocket', error);
};

// Stomp 에러 
stompClient.onStompError = (frame) => {
    console.error('Broker reported error: ' + frame.headers['message']);
    console.error('Additional details: ' + frame.body);
};

// StompJs.Client 연결
function connect() {
    stompClient.activate();
}

// StompJs.Client Close
function disconnect() {
    stompClient.deactivate();
    setConnected(false);
    console.log("Disconnected");
}

function sendName() {
    stompClient.publish({
        destination: "/app/scheduledmsg",
        body: JSON.stringify({'name': $("#name").val()})
    });
}

function showGreeting(message) {
    $("#greetings").append("<tr><td>" + message + "</td></tr>");
}

$(function () {
    $("form").on('submit', (e) => e.preventDefault());
    $( "#connect" ).click(() => connect());
    $( "#disconnect" ).click(() => disconnect());
    $( "#send" ).click(() => sendName());
});
```

### **3-3. html 코드**

```html
<!DOCTYPE html>
<html>
<head>
    <title>WebSocket</title>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@stomp/stompjs@7.0.0/bundles/stomp.umd.min.js"></script>
    <script src="/websocket/websocket.js"></script>
</head>
<body>
<div id="main-content" class="container">
    <div class="row">
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="connect">WebSocket 연결 : </label>
                    <button id="connect" class="btn btn-default" type="submit">연결</button>
                    <button id="disconnect" class="btn btn-default" type="submit" disabled="disabled">정지
                    </button>
                </div>
            </form>
        </div>
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="name">보낼 메세지는 ? </label>
                    <input type="text" id="name" class="form-control" placeholder="Your name here...">
                </div>
                <button id="send" class="btn btn-default" type="submit">Send</button>
            </form>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12">
            <table id="conversation" class="table table-striped">
                <thead>
                <tr>
                    <th>메세지</th>
                </tr>
                </thead>
                <tbody id="greetings">
                </tbody>
            </table>
        </div>
    </div>
</div>
</body>
</html>
```

***

소스는 [Hyomee GitHub](https://github.com/hyomee/hybird/tree/master/src/main/java/com/hyomee/demo/requrstandresponse/websocketStomp)에 있습니다.

***

참고 : [https://spring.io/guides/gs/messaging-stomp-websocket/](https://spring.io/guides/gs/messaging-stomp-websocket/)

참고 : [채팅 프로그램](https://velog.io/@jkijki12/STOMP-Spring-Boot) : &#x20;

[ STOMP + Spring Bootstomp + spring bootvelog.io](https://velog.io/@jkijki12/STOMP-Spring-Boot)

&#x20;
