# 4장 엔티티 매핑  
## 4.1 @Entity 
- JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 어노테이션을 필수로 붙여야한다.
- 기본 생성자는 필수이다(public 또는 protected) 
- 저장할 필드에 final을 사용해서는 안된다. 

|속성|기능|기본값|
|-----|---|---| 
|name|JPA에서 사용할 엔티티의 이름을 지정한다. 보통 기본 값이 클래스 이름을 사용한다. 만약 다른 패키지에 이름이 같은 엔티티클래스가 있다면 이름을 지정해서 충돌하지 않도록 예방한다.|클래스 이름|




## 4.2 @Table 
|속성|기능|기본값|
|-----|---|---| 
|table|매핑 테이블 이름|엔티티 이름|
|catalog|catalog 기능이 있는 데이터베이스 catalog를 매핑한다| 
|schema|schema 기능이 있는 데이터베이스에서  schema를 매칭한다|
||DDL 생성시에 유니크 제약 조건을 만든다. 2 개 이상의 복합 유니크 제약 조건을 만들 수 있다. 이 기능은 스키마 자동 생성 기능을 사용해서 DDL을 만들때만 사용된다.||
## 4.3 다양한 매핑사용 
```java

@Entity
public class Member {


    @Id
    @Column(name = "ID")
    private String id;

    @Column(name="Name")
    private String userName;

    @Column(name="AGE")
    private Integer age;

    @Enumerated(EnumType.STRING)
    private RoleType roleType;
    
    @Temporal(TemporalType.DATE)
    private Date lastModifiedDate;

    @Temporal(TemporalType.DATE)
    private Date createdDate;

    @Lob
    private String description;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Date getLastModifiedDate() {
        return lastModifiedDate;
    }

    public void setLastModifiedDate(Date lastModifiedDate) {
        this.lastModifiedDate = lastModifiedDate;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String descriptionl) {
        this.description = descriptionl;
    }

    protected Member(){

    }
```
## 4.4 데이터베이스 스키마 자동생성 
- JPA는 데이터베이스 스키마를 자동생성 하는 기능을 지원한다. 
- 클래스의 매핑정보를 보면 어떤 테이블에 어떤 컬럼을 사용하는 지 알 수 있다. 
- JPA는 이 매핑 정보와 데이터베이스 방언을 사용해서 데이터베이스 스키마를 생성한다. 
```xml
<!-- 이 속성을 추가하면 애플리케이션 실행시점에 데이터베이스 테이블을 자동으로 생성-->
<property name="hibernate.hbm2ddl.auto" value="create" />
<!-- 콘솔에 생성되는 테이블 생성 DDL을 출력 가능-->
<property name="hibernate.format_sql" value="true"/>
```
- 스키마 자동생성 기능은 개발자가 테이블을 직접 생성하는 수고를 덜 수 있다. 

|옵션|설명|
|---|---|
|create|DROP - CREATE|
|create-drop|DROP - CREATE - DROP|
|update|ALTER| 
|validate|WARNING| 
|none|None|
#### <span style="color:red"> 주의사항</span> 
- 운영 서버에서 create, creat-drop, update처럼 DDL을 수정하는 옵션을 사용하면 절대 안 된다.
- 개발서버, 개발 단계에서만 사용한다. 
     
        개발 초기 : create, update 
        CI 서버  : create, create-drop
        테스트 서버 : update, validate 
        운영 서버 : validate, none  
#### 이름 매핑 전략 
- Java 의 경우 카멜 표기법, 데이터 베이스는 언더스코어를 주로 사용한다. 
- hibernate.ejb.namaing_strategy 속성을 사용하면 이름 매핑 전략 변경이 가능하다. 
- hibernate는 org.hibernate.cfg.ImrpovedNamingStrategy 클래스를 제공한다. 
- 이 클래스는 테이블명이나 컬럼명이 생략되면 자바의 카멜 표기법을 테이블의 언더스코어 표기법으로 매핑한다. 
```xml 
<property name="hibernate.ejb.naming_strategy" value="org.hibernate.cfg.ImprovedNamingStrategy"/>
```
```yml 
# application.properties
spring.jpa.hibernate.naming.implicit-strategy=org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```


```yml
# application.yml
spring:
 jpa:
    hibernate:
      naming:
        implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

## 4.5 DDL 생성 기능 
```java 
@Column(name="NAME", nullable = false, length = 10)
private String userName;
```
- nullable 속성값을 false로 지정하면 자동 생성 DDL에 not null제약 조건을 추가 가능하다. 
- length 속성 값을 지정하면 DDL의 문자 크기 지정 가능하다. 

## 4.6 기본 키 매핑 
- 자동 생성 전략이 다양한 이유는 데이터베이스 벤더마다 지원 방식이 다르기 때문이다. 
- 키 생성 전략을 사용하려면 persistence.xml에 hibernate.id.new_generator_mappings = true 속성을 반드시 추가 해야 한다. 
- hibernate는 더 효과적이고 JPA 규격에 맞는 새로운 키 생성 전략을 개발 했는데 과거 버전과의 호환성을 위해 기본값을 false로 두었다. 
```xml
<property name="hibernate.id.new_generator_mapping" value=-"true">
```
### 4.6.1 기본키 직접 할당 전략
```java
@Id
@Column(name="id")
private String id;
```
- @Id 적용 가능 자바 타입은 다음과 같다.

        Java Primitive 
        Java Wrapper 
        String 
        java.util.Date
        java.sql.Date
        java.math.BigDecimal 
        java.math.BigInteger 
```java
Board board = new Board(); 
board.setId("id1")
em.persist(board); 

```
### 4.6.2 IDENTITY전략 
- IDENTITY는 기본 키 생성을 데이터베이스에 위임하는 전략이다. 
- MySQL, PostgreSQL, SQLServer, DB2에서 주로 사용한다. 
- IDENTITY 전략은 데이터베이스에 값을 저장하고 나서야 기본키 값을 구할 수 있을 때 사용한다. 
- IDENTITY전략을 사용하려면 @GeneatedValue의 strategy속성 값을 GenerationType.IDENTITY로 지정하면 된다. 
```java
@Entity 
public class Board{
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;
    ...

}

private static void login(EntityManager em) {
    Board board = new Board();
    em.persist(board); 
    System.out.println("board.id = " + board.getId());
}
```

#### IDENTYTY 전략과 최적화 
- IDENTITY 전략은 데이터를 데이터베이스에 INSERT한 후에 기본 키를 조회 가능하다. 
- 따라서 엔티티에 식별자 값을 할당하려면 JPA는 추가로 데이터베이스를 조회해야 한다. 
- JDBC3에 추가된 Statement.getGeneratedKeys()를 사용하면 데이터를 저장하면서 등시에 생성된 기본 키 값도 얻어올 수 있다. 
- IDENTITY 전략은 엔티티를 데이터베이스에 저장해야 식별자를 구할 수 있으므로 em.persist()를 호출하는 즉시 INSERT SQL이 데이터베이스에 저장되므로 **쓰기 지연이 동작하지 않는다**. 

### 4.6.3 SEQUENCE 전략 
- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트이다. 
- SEQUENCE 전략은 이 시퀀스를 사용해서 기본키를 생성한다. 
```sql
CREATE TABLE BOARD (
    ID BIGING NOT  NULL PRIMARY KEY,
    DATA VARCHAR(255) 

)
CREATE SEQUENCE BOARD_SEQ START_WITH 1 INCREMENT BY 1; 
```

```java
@Entity 
@SequenceGenerator(
    name = "BOARD_SEQ_GENERATOR", 
    sequenceName = "BOARD_SEQ", 
    initialValue =1, 
    allocationSize=1 
)
public class Board {
    @Id
    @GeneratedValue(strategy= GenerationType.SEQUENCE,
                    generator= "BOARD_SEQ_GENERATOR")
    private Long id; 
    
}
```
- 위 코드는 IDENTITY 전략과 같지만 내부 동작 방식은 다르다.
- SEQUENCE전략은 em.persist()를 호출할 때 먼저 데이터베이스 시퀀스를 사용해서 식별자를 조회한다. 
- 조회한 식별자를 엔티티를 할당한 후에 엔티티를 영속성 컨텍스트에 저장한다. 
- 이후 트랜잭션을 커밋해서 플러시가 일어나면 데이터베이스에 저장한다.  

|속성|기능|기본값| 
|---|---|---|
|name|식별자 생성기 이름| 필수| 
|sequenceName|데이터베이스에 등록되어 있는 시퀀스 이름|hibernate_sequence| 
|initialValue| DDL생성시에만 사용됨 시퀀스 DDL을 생성할 때 처음 시작하는 수를 지정한다| 1| 
|allocationSize|시퀀스 한 번 호출에 증가하는 수(성능최적화에 사용됨)|50|
|catalog.schema| 데이터베이스 catalog.schema이름|| 

#### <span style="color:red"> 주의사항 </span>
- SequenceGenerator.allocationSize의 기본값이 50인 것에 주의해야 한다. 
- JPA가 생성하는 데이터베이스 시퀀스는 create sequence start 1 increment by 50이므로 시퀀스를 호출할 때 마다 50씩 증가한다. 데이터베이스 시퀀스 값이 하나씩 증가하도록 설정되어있으면 이 값을 반드시 1로 설정해야 한다. 

### 4.6.4 TABLE 전략 
- TABLE 전략은 키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 컬럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략이다. 
```sql
create table MY_SEQUENCES (
    sequence_name varchar (255) not null,
    next_val bigint,
    primary key (sequence_name)
)
```
```java
@Entity
@TableGenerator(
    name = "BOARD_SEQ_GENERATOR",
    table = "MY_SEQUENCES",
    pkColumnValue = "BOARD_SEQ", allocationSize = 1)
public class Board {

    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "BOARD_SEQ_GENERATOR")
    private Long id;
}
```
- sequence_name 컬럼을 시퀀스 이름으로 사용되고 next_val 컬럼을 시퀀스 값으로 사용한다. 
- TABLE 전략은 시퀀스 대신에 테이블을 사용한다는 것만 제외하면 SEQUENCE전략과 내부 동작 방식이 같다. 
#### @TableGenetator 
|속성|기능|기본값|
|---|---|---|
|name|식별자 생성기 이름|필수|
|table|키 생성 테이블명|hibernate_sequences|
|pkColumnName|시퀀스 컬럼명|sequence_name|
|valueColumnName|시퀀스 값 컬럼명|next_val|
|pkColumnValue|키로 사용할 값 이름|엔티티 이름|
|initialValue|초기 값|0|
|allocationSize|시퀀스 한 번 호출에 증가하는 수|50|
|catalog, schema|데이터베이스 catalog, schema 이름||
|uniqueConstraints(DDL)	유니크 제약 조건을 지정할 수 있다.||	

### 4.6.5 AUTO 전략 
- 데이터베이스의 종류도 많고 기본키를 만드는 방법도 다양하다. 
- AUTO 전략의 장점은 데이터베이스를 변경해도 코드를 수정할 필요가 없다. 
- 키 생성 전략이 아직 확정되지 않은 개발 초기 단계나 프로토 타입 개발시 편리하게 사용할 수 있다. 

## 4.7 필드와 컬럼 매핑 
### 4.7.1 Column 
- @Column은 객체 필드를 테이블 컬럼에 매핑한다. 

|속성|기능|기본 값|
|---|---|---|
|name|필드와 매핑할 테이블의 컬럼 이름|객체의 필드 이름|
|insertable| 엔티티 저장 시 이 필드도 같이 저장한다. false로 설정하면 이 필드는 데이터베이스에 저장하지 않는다. false 옵션은 읽기 전용일 때 사용한다.| true |
|updatable|엔티티 수정 시 이 필드도 같이 수정한다. false로 설정하면 데이터베이스에 수정하지 않는다. false 옵션은 읽기 전용일 때 사용한다.| true|
|table|하나의 엔티티를 두 개 이상의 테이블에 매핑할 때 사용한다. 지정한 필드를 다른 테이블에 매핑할 수 있다.|현재 클래스가 매핑된 테이블|
|nullable| null 값의 허용 여부를 설정한다. false로 설정하면 DDL 생성 시에 not null 제약 조건이 붙는다.|true|
|unique| @Table의 uniqueConstraints와 같지만 한 컬럼에 간단히 유니크 제약 조건을 걸 때 사용한다.||
|columnDefinition| 데이터베이스 컬럼 정보를 직접 줄 수 있다.| 필드의 자바 타입과 방언 정보를 사용해서 적절한 컬럼 타입을 생성한다.|
|length| 문자 길이 제약 조건, String 타입에만 사용한다.|255|
|precision, scale|BigDecimal 타입 혹은 BigInteger 타입에서 사용한다. precision은 소수점을 포함한 전체 자릿수를, scale은 소수의 자릿수다.|precision = 19, scale = 2|

#### @Column 생략 
- @Column 을 생략하면 @Column 속성의 기본 값이 적용된다. 
- Java Primitive Type일 때 nullable속성에 예외가 있다. 
```java
int data1; //@Column 생략 기본 타입  
data1 integer not null //생성 DDL 

Integer data2; //Column 생략 객체 타입 
data2 integer //생성된 DDL 

@Column
int data3; //@Column 사용, 자바 기본 타입 
data3 integerb // 생성된 DDL 
```
- int data1 같은 Java 타입에는 null을 값을 입력할 수 없다. Integer data2처럼 객체 타입일 때만 null 값이 허용된다. 
- 따라서 기본 타입인 int data1을 DDL로 생성할 때는 not null 제약 조건을 추가하는 것이 안전하다. 


### 4.7.2 @Enumerated 
```java
enum RoleType {
    ADMIN, USER 

@Enumerated(EnumType,STRING)
private RoleType roleType; 

member.setRoleType(RoleType.ADMIN);
}
``` 
#### EnumType.ORDINAL
- enum에 정의된 순서대로 ADMIN은 0 USER은 1 값이 데이터베이스에 저장된다 .
- 장점 : 데이터베이스에 저장되는 데이터 크기가 작다. 
- 단점 : 이미 저장된 enum의 순서를 변경할 수 없다. 
#### EnumType.STRING 
- 이름 그대로 문자로 데이터베이스에 저장된다. 
- 장점 : 저장된 enum의 순서가 바뀌거나, enum이 추가 되어도 안전하다. 
- 단점 : 데이터베이스에 저장되는 크기가 ORDINAL에 비해서 크다. 

|속성|기능|설명| 기본 값 |
|---|---|---|---|
|value|EnumType.ORDINAL| enum 순서를 데이터베이스에 저장|default|
||EnumType.STRING| enum 이름을 데이터베이스에 저장||


### 4.7.3 @Temporal 
|속성|기능| 기본 값 |
|---|---|---|
|value|TemporalType.DATE : 날짜|TemporalType은 필수로 지정해야 한다.|
|value|TemporalType.TIME : 시간|TemporalType은 필수로 지정해야 한다.|
|value|TemporalType.TIMESTAMP : 날짜와 시간|TemporalType은 필수로 지정해야 한다.|

### 4.7.4 @Lob 
- 데이터베이스 BLOB, CLOB타입과 매핑한다. 
- @Lob 타입에는 지정할 수 있는 속성이 없지만 매핑하는 타입이 문자면 CLOB로 매핑하고 나머지는 BLOB로 매핑한다. 
- CLOB : Striing, char[], java.sql.CLOB 
- BLOB : byte[], java.sql.BLOB 
### 4.7.5 @Transient 
- 이 필드는 매핑하지 않는다. 
- 데이터베이스에 저장하지 않고 조회하지도 않느다. 객체에 어떤 값을 임시로 보관하고 싶을 때 사용한다. 

### 4.7.6 @Access 
- JPA가 엔티티 데이터에 접근하는 방식을 지정한다. 
#### 필드접근 
- AccessType.FIELD로 지정한다. 필드에 직접 접근한다. 필드 접근 권한이 private 이어도 접근할 수 있다. 
#### 프로퍼티 접근 
- AccessType.PROPERTY로 지정한다. 접근자를 사용한다. 
------ 
#### CheckList 


