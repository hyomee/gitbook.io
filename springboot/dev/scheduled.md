# Scheduled

배치 또는 데몬을 통해서 예약된 시간에 프로그램을 실행 하는 것을 간단히 스프링을 사용해서 다음과 같은 방법으로 프로그램을 할 수 있습니다.

* @Scheduled : 스프링 부트에서 @Scheduled 어노테이션을 이용하여 스케줄링
* TaskScheduler : ThreadPoolTaskScheduler를 사용해서 여러개의 예약 작업을 실행 &#x20;
* Quartz : Quartz 라이브러리를 Spring-Boot에 통합 한 spring-boot-starter-quartz 를 사용

## **1.  @Scheduled 사용**&#x20;

### 1-1. @Scheduled사용한 개발

@Scheduled는 Spring에서 생성한 한개의 Thread에서 실행이 되므로 여러개 등록이 되었을 때 하나의 job이 끝나야 다 job이 실행 됩니다.

* Application Class에 @EnableScheduling을 추가 하여  스케줄링 기능을 사용할 수 있는 상태로 만듭니다.
* @Scheduled를 실행 할 서비스를 개발 합니다.

```java
@EnableScheduling   // 스케줄링 기능활성화
@SpringBootApplication
public class HybirdApplication {  
	public static void main(String[] args) {
    	SpringApplication.run(HybirdApplication.class, args);
  	}
}

@Service
@Slf4j
public class AnnotationScheduledService {

  @Scheduled(initialDelay = 10000, fixedDelay = 10000 )
  public void scheduledJob() {
    log.debug("10 초 이후 job 다시 실행 "
            + Thread.currentThread().getName() + " : "
            + LocalDateTime.now().format(DateTimeFormatter.ofPattern("YYYY-MM-dd'T'HH:mm:ss")));
  }

  @Scheduled(cron = "0/15 * * * * *" )
  public void scheduledJobCron() {
    log.debug("cron 15초 이후 실행.. "
            + Thread.currentThread().getName() + " : "
            + LocalDateTime.now().format(DateTimeFormatter.ofPattern("YYYY-MM-dd'T'HH:mm:ss")));
  }
}


[hyomee] .... - cron 15초에 실행.. scheduling-1 : 2023-08-24T23:20:15
[hyomee] .... - 10 초 이후 job 다시 실행 scheduling-1 : 2023-08-24T23:20:18
[hyomee] .... - 10 초 이후 job 다시 실행 scheduling-1 : 2023-08-24T23:20:28
[hyomee] .... - cron 15초에 실행.. scheduling-1 : 2023-08-24T23:20:30
```

### 1-2. @Scheduled 속성

<table data-header-hidden><thead><tr><th width="182.33333333333331"></th><th></th><th></th></tr></thead><tbody><tr><td>속성</td><td>설명</td><td>예제</td></tr><tr><td>cron </td><td>Cron 표현식을 사용하여 작업을 예약  : 6자<br>- 초 : 0 -59<br>- 분 : 0 - 59<br>- 시간 :  0 - 23,<br>- 일 : 1 - 31<br>- 월 : 1 - 12<br>- 요일 : 0: 일, 1: 월, 2:화, 3:수, 4:목, 5:금, 6:토<br><br>* : 모든 조건(매시, 매일, 매주처럼 사용)을 의미<br>? : 설정 값 없음 (날짜와 요일에서만 사용 가능)<br>- : 범위를 지정할 때<br>, : 여러 값을 지정할 때<br>/ : 증분값, 즉 초기값과 증가치 설정에 사용<br>L : 마지막 - 지정할 수 있는 범위의 마지막 값 설정 시 사용 (날짜와 요일에서만 사용 가능)<br>W : 가장 가까운 평일(weekday)을 설정할 때<br># : N번째 주 특정 요일을 설정할 때 (-요일에서만 사용 가능)</td><td><br>* 매일 0시에 실행  :<br>- @Scheduled(cron = "0 0 0 * * *" )<br><br>* 10초 마다시에 실행  :<br>- @Scheduled(cron = "0/10 0 0 * * *" )<br><br>* 1시간 마다 실행  :<br>- @Scheduled(cron = "0 0 0/1 * * *" )<br><br>* 매일 오후 18시에 실행<br>@Scheduled(cron = "0 0 18 * * *") <br><br>* 매달 5, 15일 01시 실행 <br>@Scheduled(cron = "0 0 1  5,15 * *")<br><br>* 매달 마지막 23시 실행 <br>@Scheduled(cron = "0 0 23  L ? ?")<br><br>* 매일 9시~19시 사이 5분 간격으로 실행<br>@Scheduled(cron = "0 0/5 9,18 * * *") <br><br> </td></tr><tr><td>zone </td><td>ron 표현식을 사용했을 때 사용할 time zone 설정, Default 는  Local time zone</td><td></td></tr><tr><td>fixedDelay</td><td>milliseconds 단위로, 이전 Task의 종료 시점으로부터 정의된 시간만큼 지난 후 Task를 실행 ( 작업 수행 시간 포함됨 )</td><td>* 10 초 마다<br>@Scheduled(fixedDelay = 10000)</td></tr><tr><td>fixedDelayString</td><td>fixedDelay동일 문자열 </td><td>* 10 초 마다<br>@Scheduled(fixedDelayString = "10000")</td></tr><tr><td>fixedRate</td><td>milliseconds 단위로, 이전 Task의 시작 시점으로부터 정의된 시간만큼 지난 후 Task를 실행 ( 작업 수행 시간과 관계 없음 )</td><td>* 10 초 마다<br>@Scheduled(fixedRate = 10000)</td></tr><tr><td>fixedRateString</td><td>fixedRate 동일 문자열</td><td>* 10 초 마다<br>@Scheduled(fixedRateString = "10000")</td></tr><tr><td>initialDelay</td><td>초기 지연 시간 설정 </td><td>* 3초 이후 5초 마다 실행 <br>@Scheduled(fixedRate = 5000, <br>                      initialDelay = 3000)</td></tr><tr><td>initialDelayString</td><td>initialDelay 동일 문자열 </td><td>* 3초 이후 5초 마다 실행 <br>@Scheduled(fixedRate = 5000, <br>                      initialDelayString = "3000")</td></tr><tr><td>timeUnit</td><td>스케줄 시간 단위 설정, default : MILLISECONDS</td><td>TimeUnit.NANOSECONDS(1L),<br>TimeUnit.MICROSECONDS(1000L),<br>TimeUnit.MILLISECONDS(1000000L),<br>TimeUnit.SECONDS(1000000000L),<br>TimeUnit.MINUTES(60000000000L),<br>TimeUnit. HOURS(3600000000000L),<br>TimeUnit. DAYS(86400000000000L);</td></tr></tbody></table>

@Scheduled의 단점인 하나의 Thread에서 실행 하는 것을 Thread Pool을 만들어서 job을 실행 합니다

* SchedulingConfigurer을 상속 받아 config를 구성 합니다.

```java
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {

  @Value("${scheduler.poolSize:10}")
  private int POOL_SIZE;

  /**
   * @Scheduled 에서 실행 되는 job을 각각의 Thread에 실행 하기 위한 SchedulerConfig
   * @param taskRegistrar the registrar to be configured.
   */
  @Override
  public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
    final ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
    threadPoolTaskScheduler.setPoolSize(POOL_SIZE);
    threadPoolTaskScheduler.setThreadNamePrefix("hybird-scheduled-task-pool-");
    threadPoolTaskScheduler.initialize();

    taskRegistrar.setTaskScheduler(threadPoolTaskScheduler);
  }

}


[hyomee] .... - 10 초 이후 job 다시 실행 hybird-scheduled-task-pool-1 : 2023-08-24T23:44:57
[hyomee] .... - cron 15초 이후 실행.. hybird-scheduled-task-pool-2 : 2023-08-24T23:45:00
[hyomee] .... - 10 초 이후 job 다시 실행 hybird-scheduled-task-pool-1 : 2023-08-24T23:45:07
```

&#x20;

그외 방법에 대해서는 다음을 참고 하세요

참고 : [tutorials/spring-scheduling at master · eugenp/tutorials (github.com)](https://github.com/eugenp/tutorials/tree/master/spring-scheduling)

참고 : [Scheduling in Jakarta EE | Baeldung](https://www.baeldung.com/scheduling-in-java-enterprise-edition)&#x20;

참고 : [https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/boot-features-quartz.html](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/boot-features-quartz.html)    &#x20;
