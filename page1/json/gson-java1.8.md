---
description: Gson - Json Serialization & Deserialization
---

# Gson Java1.8 직렬화/역직열화

## 문제점

Gson을 사용해서 자바 객체로 변환하는 과정에서 다음과 같은 오류가 발생하는 경우 해결 방안입니다.

```
Unable to make field private final java.time.LocalDate java.time.LocalDateTime.date accessible: 
module java.base does not "opens java.time" to unnamed module
```

## **1. 원인**

Gson은 Java 객체를 json으로 json을 Java 객체로 변환해 주는 라이브러리로 변환 과정에 위와 같은 오류가 나는 이유는 LocalDate 또는 LocalDateTime 과 같은 Java 8 날짜 시간 클래스에 속하는 Java 클래스를 직렬화하거나 역직렬화할 때 발생 하는 오류로 public으로 선언한 필드에 대해서 직렬화/역직렬화를 제공하는데 private을 선언한 필드에 대해서 하는 경우 접근을 하지 못하므로 발생합니다.

### **- 직렬화/역직렬화 란** &#x20;

* Json 직렬화(serialization) : Java 객체를 Json 문법으로 변환하는 것&#x20;
* 역직렬화(deserialization) : Json 문법으로 작성된 문자열을 Java 객체로 변환하는 것

## **2. 해결방안**

Gson 생성 시점에 Gson 속성 중 registerTypeAdapter을 만들어서 설정하면 됩니다. 즉 JsonSerializer , JsonDeserializer  인터페이스를 구현하는  구현체를 만들어서 Gson의 registerTypeAdapter로 설정하여 직렬화 및 역직렬화하게 하면 됩니다. 예제로 2가지 방법을 소개합니다.

* &#x20;JsonSerializer , JsonDeserializer을 직접 상속받는 방법&#x20;
* &#x20;TypeAdapter를 상속하는 방법&#x20;

### 2-1. JsonSerializer , JsonDeserializer을 직접 상속받는 방법

#### 2-1-1. 자바 1.8 Java 8 날짜 시간 클래스에 있는 LocalDateTime, LocalDate, LocalTime의 private 객체에 접근하여 직렬화/역직렬화 수행하는 클래스를 생성합니다.&#x20;

```java
// 날짜/시간 Type에 따라 사용할 상수 선언 
public final static String PATTERN_DATE = "yyyy-MM-dd";
public final static String PATTERN_TIME = "HH:mm:ss";
public final static String PATTERN_DATETIME = String.format("%s %s", PATTERN_DATE, PATTERN_TIME);

// LocalDateTime에 대한 직렬화/역직렬화
public class LocalDateTimeTypeAdapter implements 
	JsonSerializer<LocalDate>, JsonDeserializer<LocalDate> {

// 변환 날짜 포맷 선언 
    private final DateTimeFormatter formatter = DateTimeFormatter.ofPattern(CommonConstant.PATTERN_DATETIME);

// 직렬화 
    @Override
    public JsonElement serialize(final LocalDate date,
    							 final Type typeOfSrc,
                                 final JsonSerializationContext context) {
        return new JsonPrimitive(date.format(formatter));
    }

// 역직렬화
    @Override
    public LocalDate deserialize(final JsonElement json, 
    							 final Type typeOfT,
                                 final JsonDeserializationContext context) throws JsonParseException {
        return LocalDate.parse(json.getAsString(), formatter);
    }
}
```

#### 2-1-2.  Spring @Configuration을 통해서 Bean 주입하는 하기 위해 구성 요소로 등록합니다.

```
@Configuration
public class Gsonfiguration {

    @Bean
    public Gson gson() {

        return new GsonBuilder()
                .disableHtmlEscaping()
                .serializeNulls()
                .setLenient()
                .registerTypeAdapter(LocalDate.class, new LocalDateTypeAdapter())
                .registerTypeAdapter(LocalDateTime.class, new LocalDateTimeTypeAdapter())
                .registerTypeAdapter(LocalTime.class, new LocalTimeTypeAdapter())
                .create();
    }

}
```

* disableHtmlEscaping : Html Escape 되지 않고 그대로 직렬화하는 것을 Html Format로 이쁘게 함&#x20;
* serializeNulls :  Gson은 null인 필드값은 직렬화 대상에서 제외시키는데 필드값이 null인 경우에도 직렬화에 포함
* setLenient : 기본적으로 Gson은 엄격하며 [RFC 4627에](http://www.ietf.org/rfc/rfc4627.txt) 지정된 대로 JSON만 허용하는 파서가 허용하는 내용을 자유롭게 하도록 하는 기능&#x20;
* registerTypeAdapter : 사용자 지정 직렬화 또는 역직렬화를 위해 Gson을 구성하는 것으로 . JsonSerializer , JsonDeserializer  인터페이스를 구현하는  사용자 정의 구현체 만들어  type이을 재정의 함

### 2-2.  TypeAdapter를 상속하는 방법&#x20;

3-1과 구현 과정은 동일하며 예제는 다음과 같습니다,

```java
class LocalDateTimeAdapter extends TypeAdapter<LocalDateTime> {
	DateTimeFormatter format = DateTimeFormatter.ofPattern(CommonConstant.PATTERN_DATETIME);

	@Override
	public void write(JsonWriter out, LocalDateTime value) throws IOException {
		if(value != null)
			out.value(value.format(format));
        }

	@Override
	public LocalDateTime read(JsonReader in) throws IOException {
		return LocalDateTime.parse(in.nextString(), format);
	}
}
```

### 3.  참고&#x20;

#### 3-1.  [Gson Doc  ( gson v2.10.1  : 2023.09 최신 버전 ) ](https://www.javadoc.io/doc/com.google.code.gson/gson/latest/com.google.gson/com/google/gson/GsonBuilder.html).

#### 3-2.  [com.google.gson module summary](https://www.javadoc.io/doc/com.google.code.gson/gson/latest/com.google.gson/module-summary.html)

\


\
