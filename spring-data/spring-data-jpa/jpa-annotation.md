# JPA Annotation

* 객체와 테이블 매핑 : @Entity, @Table
* 기본키 매핑 : @Id, @GeneratedValue, @SequenceGenerator, @TableGenerator
* 필드와 컬럼 매핑 : @Column, @Enumerated, @Temporal, @Lob, @Transient

## 1. 기본 어노테이션

<table data-header-hidden><thead><tr><th width="185"></th><th width="101"></th><th></th></tr></thead><tbody><tr><td>어노테이션</td><td>적용</td><td>기능</td></tr><tr><td>@Entity</td><td>클래스</td><td>JPA 에게 TABLE 매핑 -> 엔티티 클래스</td></tr><tr><td>@Table</td><td>클래스</td><td>@Table 생략시 클래스 이름이 테이블 이름· name : 테이블 이름· catalog : 테이블 카테고리· schema : 테이블 스키마· uniqueConstraints : 컬럼값 유니크 제약 조건· indexes : 인덱스 생성</td></tr><tr><td>@Access</td><td>클래스, 필드</td><td>· AccessType.FIELD : 필드에 직접 접근· AccessType.PROPERTY : getter를 통해 접근</td></tr><tr><td>@Id</td><td>필드</td><td>테이블의 기본키(PK) - 식별자 필드@GeneratedValue 으로 키 생성 전략<br><br>strategy=GenerationType.TABLE,<br>strategy=GenerationType.SEQUENCE,<br>strategy=GenerationType.IDENTITY,<br>strategy=GenerationType.UUID,<br>strategy=GenerationType.AUTO</td></tr><tr><td>@Column</td><td>필드</td><td>필드를 컬럼에 매핑<br><br>· name : 컬럼 이름· unique : 유니크 여부 (DDL)· nullable : null 허용 여부 (DDL)· insertable : 추가 가능 여부 (DDL, DML)· updatable : 수정 가능 여부 (DDL, DML)· table : 테이블 이름 (하나의 엔티티를 두 개 이상의 테이블에 매핑할 때)· columnDefinition : 데이터베이스 컬럼 정보 직접 부여 (DDL)· length : 컬럼 사이즈 (default:255) (DDL)· precision : 소수 정밀도 (default:0) (DDL)· scale : 소수점 이하 자리수 (default:0) (DDL)</td></tr><tr><td>@GeneratedValue</td><td>필드</td><td>식별키 생성· strategy : 식별키 전략 지정 (default:AUTO)<br><br>strategy=GenerationType.TABLE,<br>strategy=GenerationType.SEQUENCE,<br>strategy=GenerationType.IDENTITY,<br>strategy=GenerationType.UUID,<br>strategy=GenerationType.AUTO</td></tr><tr><td>@SequenceGenerator</td><td>클래스, 필드</td><td>테이블 기본키 생성 전략을 시퀀스로 이용할 경우 시퀀스에 정보를 지정합니다.· name : 식별자 이름· sequenceName : 데이터베이스안에 정의된 실제 시퀀스 이름· initialValue : DDL 생성시에만 사용되며, 시퀀스 DDL 생성시 초기값· allocationSize : 시퀀스 한 번 호출에 증가하는 수· catalog : 데이터베이스 catalog· schema : 데이터베이스 schema</td></tr><tr><td>@TableGenerator</td><td>클래스, 필드</td><td>테이블 기본키 생성 전략 :  채번 테이블을 이용할 경우  · name : 식별자 이름· table : 키생성 테이블명· pkColumnName : 시퀀스 컬럼명· valueColumnName : 시퀀스 값 컬럼명· pkColumnValue : 키로 사용할 값 이름· initialValue : 초기 값, 마지막으로 생성된 값이 기준· allocationSize : 시퀀스 한 번 호출에 증가하는 수· catalog : 데이터베이스 catalog· schema : 데이터베이스 schema· uniqueConstraints(DDL) : 유니크 제약 조건 지정</td></tr><tr><td>@Index</td><td>클래스</td><td>인덱스 지정· name : 인덱스 이름· columnList : 인덱스를 지정할 컬럼들· unique : 중복값 허용 여부 (default:false[중복 허용])</td></tr><tr><td>@Enumerated</td><td>필드</td><td>자바 enum 타입을 매핑· EnumType.ORGINAL : enum 순서 · EnumType.STRING : enum 이름 </td></tr><tr><td>@Temporal</td><td>필드</td><td>날짜 타입을 매핑할 때 사용합니다.· TemporalType.DATE : 날짜, DB date  · TemporalType.TIME : 시간, DB time  · TemporalType.TIMESTAMP : 날짜와 시간 DB timestamp 타입 </td></tr><tr><td>@Lob</td><td>필드</td><td>DB BLOB, CLOB 타입과 매핑· 필드 타입이 문자열이면 CLOB 매핑· 그 외 자료형이면 BLOB 매핑</td></tr><tr><td>@Transient</td><td>필드</td><td>DB에 저장하지도 조회하지도 않음</td></tr><tr><td>@CreationTimestamp</td><td>필드</td><td>INSERT 쿼리가 발생할 때, 현재 시간을 자동으로 만들어 저장</td></tr><tr><td>@UpdateTimestamp</td><td>필드</td><td>UPDATE 쿼리가 발생할 때, 현재 시간을 자동으로 민들어 저장</td></tr></tbody></table>

***

## 2. 연관 관계 매핑 :&#x20;

@OneToOne, @OneToMany, @ManyToOne, @ManyToMany, @Joincolumn

자세한 사용방법은 [JPA연관 관계](https://hyomee.gitbook.io/develop/jpa-and-query-dsl/jpa/jpa-2)를 참조하세요.
