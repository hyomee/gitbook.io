# Static 언제 사용

static 은 정적이라는 의미로 하나의 JVM or WAS에서는 같은 주소에 존재하는 값으로 GC 대상이 아니다.

다음과 같은 경우에 사용

1. 자주 사용하고 절대 변경되지 않는 변수는 final static 선언&#x20;
2. 설정 파일 정보
3. 코드성 데이터 ( db에서 한번 읽어와 보관 ) \
   \- in-memory 사용 권장 ( memcahed, EhCache ... )
4. 여러 곳에 변경이 발생하는 곳에는 사용하지 말아야 한다. ( 죽음 ... : 예상치 못한 오류 : 멀티 환경 )\
   \- Vector, ArrayList 사용시 OutOfMemoryError 발생&#x20;

####



