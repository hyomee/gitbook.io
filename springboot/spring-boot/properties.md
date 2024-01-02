# Properties

애플리케이션을 실행 하면서 필요한 정보를 외부 파일에 정의 후 필요 시점에 사용할 수 있는데 스프링 부트에서는 application.properties 또는 application.yml 파일에 정의 하는 방법과 사용자 정의 파일을 만들어서 사용하는 방법을 모두 제공한다.

애플리케이션 외부 설정 파일의 참조 순서는 다음과 같다.

> properties을 활성화 하기 위해서는 spring.profiles.active 속성으로 전달 되어야 한다.

1. 명령행 인수
2. 패키징된 애플리케이션 외부의 application-{profile}.properties
3. 패키징된 애플리케이션 외부의 application.properties
4. 패키징된 애플리케이션 내부의 application-{profile}.properties
5. 패키징된 애플리케이션 내부의 application.properties

> @Value(“${속성명”})을 사용해서 읽어올 수 있다.

> \-jar test.jar spring.profiles.active=act로 실행을 하면 application.properties파일 보다 application-act.properties의 속성이 동일 이름을 가지고 있다면 우선 순위가 높다.
