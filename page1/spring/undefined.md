# 특정 시간에 서비스 호출

## 1. 요구사항

특정 시간에 특정 서비스를 호출하여 작업을 실행 해야 합니다.

## 2. 해결방안

배치 또는 데몬을 통해서 예약된 시간에 프로그램을 실행 하는 것을 간단히 스프링을 사용해서 다음과 같은 방법으로 프로그램을 할 수 있습니다.

* @Scheduled : 스프링 부트에서 @Scheduled 어노테이션을 이용하여 스케줄링
  * [스프링 부트의 Scheduled에 대해서는 여기를 참조 하면 됩니다.](https://hyomee.gitbook.io/develop/springboot/scheduled)
* TaskScheduler : ThreadPoolTaskScheduler를 사용해서 여러개의 예약 작업을 실행 &#x20;
* Quartz : Quartz 라이브러리를 Spring-Boot에 통합 한 spring-boot-starter-quartz 를 사용

