# Spring Filter, Interceptor, AOP

<figure><img src="../../.gitbook/assets/image (180).png" alt=""><figcaption></figcaption></figure>

* javax.servlet.Filter.ServletRequestListener : 요청 시작과 요청 종료시의 타이밍에서 어떤 작업을 수행
* &#x20;javax.servlet.Filter : Servlet, JSP, 정적 콘텐츠 등의 Web 리소스에 대한 액세스 전후에 공통 작업을 수행&#x20;
* HandlerInterceptor : Spring MVC의 Handler의 호출 전후에 일반적인 작업을 수행&#x20;
* @ControllerAdvice : Controller 전용의 특수한 메소드 (@InitBainder메소드,@ModelAttribute메소드,@ExceptionHandler메소드)를 복수의 Controller에서 공유&#x20;
* Spring AOP (AspectJ) : Spring의 DI 컨테이너에서 관리되는 Bean의 public 메소드 호출 전후에 일반적인 작업을 수행

## 1. Filter, Interceptor, AOP 흐름

* Interceptor와 Filter는 Servlet 단위에서 실행된다. <> 반면 AOP는 메소드 앞에 Proxy패턴의 형태로 실행 됨&#x20;
* 요청이 들어오면 Filter → Interceptor → AOP → Interceptor → Filter 순으로 거치게 된다&#x20;
* &#x20;Interceptor :&#x20;

## 2. Filter

* 요청과 응답 사이에서 Data 정체 역할&#x20;
* 스프링 컨텍스트 외부에 존재하여 스프링과 무관한 자원에 대해 동작 - 예) 인코딩, XSS 방어 등 :&#x20;
* Fliter 실행 메소드&#x20;
  * \- init() - 필터 인스턴스 초기화&#x20;
  * \- doFilter() - 전/후 처리&#x20;
  * \- destroy() - 필터 인스턴스 종료&#x20;

## 3. Interceptor&#x20;

* 인터셉터는 스프링의 DistpatcherServlet이 컨트롤러를 호출하기 전, 후로 끼어들기 때문에 스프링 컨텍스트(Context, 영역) 내부에서 Controller(Handler)에 관한 요청과 응답에 대해 처리
* &#x20;스프링의 모든 빈 객체에 접근&#x20;
* HttpServletRequest, HttpServletResponse를 파라미터로 사용 : 인터셉터는 여러 개를 사용할 수 있고 로그인 체크, 권한체크, 프로그램 실행시간 계산작업 로그확인 등의 업무처리&#x20;
* 인터셉터의 실행 메서드&#x20;
  * preHandler() - 컨트롤러 메서드가 실행되기 전&#x20;
  * postHanler() - 컨트롤러 메서드 실행직 후 view페이지 렌더링 되기 전&#x20;
  * afterCompletion() - view페이지가 렌더링 되고 난 후&#x20;

## 4. AOP&#x20;

* 메소드 전후의 지점에 자유롭게 설정&#x20;
* Advice의 경우 JoinPoint나 ProceedingJoinPoint 등을 활용&#x20;
* AOP의 포인트컷&#x20;
  * @Before: 대상 메서드의 수행 전&#x20;
  * @After: 대상 메서드의 수행 후&#x20;
  * @After-returning: 대상 메서드의 정상적인 수행 후&#x20;
  * @After-throwing: 예외발생 후&#x20;
  * @Around: 대상 메서드의 수행 전/후
