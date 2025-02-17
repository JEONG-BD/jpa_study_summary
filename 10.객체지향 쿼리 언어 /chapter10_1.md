# 10장 객제지향 쿼리 언어 
## 10.1 객체지향 쿼리 소개
-  EntityManager.find() 메소드를 사용하면 식별자로 엔티티를 하나 조회할 수 있다. 
- 이렇게 조회한 엔티티에 객체 그래프 탐색을 사용하면 연관된 엔티티를 찾을 수 있지만 이 기능만으로는 애플리케이션을 개발하기 어렵다.  
- 예를 들어 나이가 30살 이상인 회원을 모두 검색하고 싶다면 좀 더 현실적이고 복잡한 방법이 필요하다. 데이터는 데이터베이스에 있으므로 SQL로 필요한 내용을 최대한 걸러서 조회해야 한다. 
- ORM을 사용하면 데이터베이스 테이블이 아닌 엔티티 객체를 대상으로 개발하므로 테이블이 아닌 엔티티 객체를 대상으로 하는 방법이 필요하다.다. 

### 10.1.1 JPQL 소개 
- JPQL(Java Persistence Query Language)은 객체를 조회하는 객체지향 쿼리이다. 
- JPQL은 SQL을 추상화해서 특정 베이스에 의존하지 않고 SQL보다 간결하다. 
```java
@Entity(name="Member")
public class Member {
    @Column(name="name")
    private String username; 


}
String jpql = "select m from Member as m where m.username = 'kim'";
List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();

```
### 10.1.2 Criteria 쿼리 소개 
- Criteria 는 JPQL을 생성하는 빌더 클래스이다. Criteria의 장점은 문자가 아닌 프로그래밍 코드로 JPQL을 작성할 수 있다는 것이다. 
- 컴파일 시점에 오류를 발견할 수 있다. 
- IDE를 사용하면 코드 자동완성을 지원한다. 
- 동적 쿼리를 작성하기 편하지만, 이 장점들을 상쇄할 정도로 복잡하고 장황하다. 

### 10.1.3 QueryDSL 
- Criteria 처럼 JPQL빌더 역할을 한다. 
- QueryDSL의 장점은 코드 기반이 단순하고 사용하기 쉽다.

### 10.1.4 네이티브 SQL 소개 
- JPA에서 SQL을 직접 사용할 수 있는 기능을 지원하는데 이것을 네이티브 SQL 이라도 한다. 
- 네이티브 SQL의 단점은 특정 데이터베이스에 의존하는 SQLdmf 작성해야 한다는 점이다. 
### 10.1.5 JDBC 직접 사용, 마이바티스 같은 SQL메퍼 프레임워크 사용 
- JDBC 커넥션을 직접 접근하고 싶으면 JPA는 JDBC 커넥션을 획득하는 API를 제공하지 않으므로 JPA구현체가 제공하는 방법을 사용 해야 한다. 
- JDBC나 마이바티스를 JPA를 함께 사용하면 영속성 컨텍스트를 적절한 시점에 강제로 플러시해야 한다. 
- JDBC를 직접 사용하든 마이바티스 같은 SQL매퍼와 사용하든 모두 JPA를 우회해서 사용해야 하는데 JPA를 우회하는 SQL에 대해서는 JPA가 전혀 인식하지 못하고 최악의 시나리오는 데이터 무결성을 훼손할 가능성이 있다. 
- JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트를 수동으로 플러시해서 데이터베이스와 영속성을 컨텍스트를 동기화하면 된다. 

## 10.2 JPQL 
### 10.2.1 기본 문법과 쿼리 API 
- JPQL도 SQL과 비슷하게 SELECT, UPDATE, DELETE문을 사용할 수 있다. 
#### SELECT 문 
```java
SELECT m FROM Member AS mwhere m.username = "hello"
```
- 대소문자를 구분하지만, JPQL 키워드는 대소문자를 구분하지 않는다. 
- 엔티티명을 지정하지 않으면 클래스명을 기본값으로 사용한다. 
- 별칭은 필수 이다.
#### Type query 
- JPQL을 실행하려면 쿼리 객체를 만들어야 한다. TypeQuery와 Query가 있는데 반환할 타입을 명확하게 지정할 수 있으면 TypeQuery객체를 사용하고 지정할 수 없으면 Query를 만들면 된다. 
```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member.m", Member.class); 

List<Member> resultList = query.getResultList(); 

for(Member member : resultList){
    System.out.println("member = " + member);
}

Query query = em.createQuery("SELECT m.username, m.age from Member m"); 

List resultList = query.getResultList(); 

for(Object o: resultList){
    Object[] result = (Object[])o;
}
```
- Query 객체는 SELECT 절의 조회대상이 둘 이상이면 Object[] 를 하나면 Object를 반환한다. 
#### 결과 조회 
- query.getResultList() 결과 반환한다. 만약 결과가 없으면 빈 컬렉션을 반환한다. 
- query.getSingleResult() 결과가 정확히 하나일 때 사용한다. 
- 결과가 없으면 javax.persistence.NoResultException예외를 반환한다. 
- 결과가 1개 보다 많으면 javax.persistence.NonUniqueResultException예외가 발생한다. 

### 10.2.2 파라미터 바인딩 
- JDBC는 위치 기준 파라미터 바인딩을 지원하지만 JPQL은 이름 기준 파라미터 바인딩도 지원한다. 
#### 이름 기준 파라미터 
```java
String usernameParam = "User1" ;
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class); 
query.setParameter("username", username); 
List<Member>resultList = query.getResultList();  

```
- JPQL API는 대부분 메소드 체인 방식으로 설계되어 있어서 아래와 같이 작성하는 것이 가능하다. 
```java
String usernameParam = "User1" ;
List<Member>resultList = em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class)
.setParameter("username",username)
.getResultList();  
```

#### 위치 기준 파라미터 
```java
String usernameParam = "User1" ;
List<Member>resultList = em.createQuery("SELECT m FROM Member m where m.username = ?1", Member.class).setParameter(1,username).getResultList();  
```
#### 주의
- JPQL을 수정해서 파라미터 바인딩 방식을 사용하지 않고 직접 문자열을 더해 만들어 넣으면 악의적인 사용자에 의해 SQL인젝션 공격을 당할 수 있고, 성능 이슈도 있다. 따라서 파라미터 바인딩 방식은 선택이 아닌 필수이다.

### 10.2.3 프로젝션  
- SELECT 절에 조회할 대상을 지정하는 것을 프로젝션이라 하고 SELECT {프로젝션 대상} FROM으로 대상을 선택한다. 
- 프로젝션 대상으로는 엔티티 임베디드 타입, 스칼라 타입이 있다. 
#### 엔티티 프로젝션 
```java
SELECT m FROM Member m; 
SELECT m.team FROM Member m; 
```
#### 임베디드 타입 프로젝션 
- 임베디드 타입은 조회의 시작점이 될 수 없다. 
```java
// String query = "SELECT a FROM Address a"; 

String query = "SELECT o.address FROM Order o" 
List<Address> address = em.createQuery(query, Address.class).getResultList();
```
- 임베디드 타입은 엔티티 타입이 아닌 값 타입이다. 때문에 임베디드 타입은 영속성 컨텍스트에서 관리 되지 않는다. 


#### 스칼라 타입 프로덕션 
```java
List<String> username = em.createQuery("SELECT username FROM Member m", String.class).getResultList();
```

#### 여러 값 조회  
- 프로젝션에 여러 값을 선택하면 TypeQuery를 사용할 수 없고 대신에 Query를 사용해야 한다. 
```java
Query query = em.createQuery("SELECT m username. m.age, FROM Member m"); 
List resultList = query.getResultList(); 

Iterator iterator = resultList.iterator(); 
while(iterator.hasNext()){
    Object[]row = (Object[]) iterator.next();
    String username = (String)row[0];
    int age = (Integer) row[1]; 

}
for (Object[] row : resultList){
    String username = (String) row[0]; 
    Integer age = (Integer) row[1];
}
```

```java
List<Object[]> resultList = em.createQuery("SELECT o.member, o.orderAmount, o.product FROM Order o")
.getResultList();

for(Object[] row: resultList){
    Membber = (Member)row[0];
    Product = (Product)row[1];
    int orderAmount = (Integer)row[2];
}
```
#### NEW 명령어 
- 위의 코드에서 username, age 두 필드를 프로젝션해서 타입을 지정할 수 없으므로 TypeQuery를 사용할 수 없어서 Object[]를 반환받았다. 
- 실제 개발시에는 Object[]를 직접 사용하지 않고 UserDTO처럼 의미 있는 객체로 변환해서 사용하는 것을 권장한다. 
```java
public class UserDTO {

    private String username;
    private int age;

    public UserDTO(String username, int age){
        this.username = username; 
        this.age = age ;
    }

}

TypedQuery<UserDTO> query = 
    em.createQuery("SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m", 
    UserDTO.class);

List<UserDTO> resultList = query.getResultList();
```
- SELECT 다음에 NEW 명령어를 사용하면 반환받을 수 있는 클래스를 지정할 수 있는데 이 클래스의 생성자에 JPQL 조회 결과를 넘겨줄 수 있다. 
- NEW 명령어를 사용할 때는 패키지 명을 포함한 전체 클래스 명을 입력하는 것과 순서, 타입이 일치하는 생성자가 필요하다. 

### 10.2.4 페이징 API 
JPA는 페이징을 다음 두 API로 추상화하였다.
- setFirstResult(int startPosition) : 조회 시작 위치(0부터 시작한다)
- setMaxResults(int maxResult) : 조회할 데이터 수
```java 
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class);

query.setFirstResult(10); // 11번부터 시작
query.setMaxResults(20); // 11번~30번 데이터 조회
query.getResultList();
```
- 데이터베이스 마다 다른 페이징 처리를 같은 API로 처리할 수 잇는 것은 데이터베이스 방언 덕분이다. 
- 페이징 SQL을 더 최적화 하고 싶다면 JPA 가 제공하는 페이징 API가 아닌 네이티브 SQL을 직접 사용해야 한다. 

### 10.2.5 집합과 정렬 
|함수|설명|
|--|--|
|COUNT|결과수 반환 Long|
|MAX, MIN|최대값과 최소값 반환|
|AVG|평균 반환 Double|
|SUM|합 반환|

#### 집합함수 사용시 참고사항 
- NULL값은 무시하므로 통계에 집계되지 않는다. 
- 값이 없는데 집합 함수를 쓰면 NULL이 된다. 
- DISTINCT를 집합 함수 안에 사용해서 중복된 값을 제거하고 나서 집합을 구할 수 있다. 
- DOSTINCT를 COUNT에 사용할 때 임베디드 타입은 지원하지 않는다. 

#### GROUP BY, HAVING 
- 통계 쿼리는 보통 전체 데이터를 기준으로 처리하므로 실시간으로 사용하기에는 부담이 많다. 결과가 아주 많다면 통계 결과만 저장하는 테이블을 별도로 만들어 두고 사용자가 적은 새벽에 통계 쿼리를 실행해서 그 결과를 보관하는 것이 좋다. 
```java
groupby_절 ::= GROUP BY{단일값 | 별칭 } + having_절 ::= HAVING 조건식
```

#### ORDER BY 
- 결과를 정렬할 때 사용한다. 
- ASC 오름 차순, DESC 내림차순 

### 10. 2.6 JPQL 조인 
SQL 조인과 기능은 같고 문법만 약간 다르다. 
#### 내부 조인 
```java
String teamName = "팀A" 
String query = "SELECT m FROM Member m INNER JOIN m.team t " 
                + "WHERE t.name = :teamName";

List<Member> members = em.createQuery(query, Member.class).
    setParameter("teamName", teamName)
    .getResultList();

SELECT m
FROM Member m INNER JOIN m.team t
where t.name = :teamName

//FROM Member m JOIN team t 문법 오류 

SELECT m, t 
FROM Member m JOIN m.team t 
where t.name = :teamName

List<Object[]> result = em.createQuery(query).getResultList();

for(Object[] row : result){
    Member member = (Member)row[0];
    Team team (Team)row[1];
}
```

#### 외부 조인 
```java
SELECT m
FROM Member m LEFT JOIN m.team t
```

#### 컬렉션 조인 
- 일대다 관계나 다대다 관계처럼 컬렉션을 사용하는 곳에 조인하는 것을 컬렉션 조인이라고 한다. 
- [회원 → 팀] 조인은 단일 값 연관 필드를 사용하고, [팀 → 회원] 조인은 컬렉션 값 연관 필드를 사용한다.
```java
SELECT t, m
FROM Team t LEFT JOIN t.members m
```

#### 세타조인
- WHERE 절을 사용해서 세타 조인을 할 수 있다. (세타 조인은 내부 조인만 지원한다.)
```java
select count(m) from Member m, Team t
where m.username = t.name
``` 
#### JOIN ON절(JPA 2.1) 
- JPA 2.1부터 조인할 때 ON절을 지원한다. ON절을 사용하면 조인 대상을 필터링 하고 조인할 수 있다. 내부 조인에 ON절은 WHERE절 사용 결과와 같으므로 보통 ON절은 외부 조인에서만 사용한다. 
```java
select m, t from Member m
left join m.team t on t.name = 'A'
``` 

### 10.2.7 페치 조인 
- 페치조인은 SQL에서 이야기 하는 조인의 종류가 아니고 JPQL에서 성능 최적화를 위해 제공하는 기능으로 연관된 엔티티나 컬렉션을 한번에 같이 조회하는 기능인데 join 명령어로 사용할 수 있다. 
#### 엔티티 페치 조인 
```java
// 회원과 연관된 팀을 함께 조회하는 JPQL 
select m
from Member m join fetch m.team
```
- 일반적인 JPQL조인과는 다르게 별칭이 없는데 페치 조인은 별칭을 사용할 수 없다. 
```java
String jpql = "select from Member m join fetch m.team"; 
List<Member> members = em.createQuery(jpql, Member.class); 
for (Member member :members){
    ...
}
```
- 회원과 팀을 지연로딩 설정했다고 가정했을 때 페치조인을 사용했으므로 연관된 팀 엔티티는 프록시가 아니라 실제 엔티티다. 
- 연관된 팀을 사용해도 지연로딩이 일어나지 않는다. 

#### 컬렉션 페치 조인 
```java
// JPQL
select t
from Team t join fetch t.members
where t.name = '팀A'

// SQL
SELECT T.*, M.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID = M.TEAM_ID
WHERE T.NAME = '팀A'
```
- 일대다 조인은 결과가 증가할 수 있지만 다대일은 결과가 증가하지 않는다. 

#### 페치 조인과 DISTINCT 
- JPQL의 DISTINCT 명령어는 SQL에 DISTINCT를 추가하고 애플리케이션에서 한번 더 중복을 제거한다. 
#### 페치 조인과 일반 조인의 차이 
- JPQL은 결과를 반환할 때 연관관계 까지 고려하지 않는다. 단지 SELECT에서 정한 엔티티만 조회할 뿐이다. 
- 컬렉션을 지연로딩 설정하고 프록시나 아직 초기화하지 않는 컬렉션 레퍼를 반환한다. 
- 즉시 로딩을 설정하면 회원 컬렉션을 즉시 로딩하기 위해 쿼리를 한번 더 실행한다. 
- 페치 조인을 사용하면 연관된 엔티티도 함께 조회한다. 

#### 페치 조인의 특징 
- 페치 조인을 사용하면 SQL 한 번으로 연관 엔티티를 함께 조회가 가능해서 SQL호출 횟수를 줄여 성능 최적화가 가능하다. 
- 엔티티에 직접 적용하는 로딩 전략은 애플리케이션 전체에 영향을 미치므로 글로벌 로딩 전략이라고 부른다. 
- 페치 조인은 글로벌 로딩 전략보다 우선한다. 따라서 글로벌 로딩 전략을 지연로딩으로 설정해도 JPQL에서 페치 조인을 사용하면 페치 조인을 적용해서 함께 조회한다. 
- **글로벌 로딩 전략은 지연 로딩을 사용하고 최적화 시에 페치조인을 사용하는 것을 권장한다**. 

#### 페치조인의 한계 
- 페치 조인 대상에는 별칭을 줄 수 없다.
- 둘 이상의 컬렉션을 페치할 수 없다.
- 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.
- 단일 값 연관 필드는 페치 조인을 사용해도 페이징 API를 사용할 수 있다. 

### 10.2.8 경로 표현식 
경로 표현식이란 .(점)을 찍어서 객체 그래프를 탐색하는 것이다. 
```java
select m.username // 경로 표현식 사용
from Member m 
   join m.team t // 경로 표현식 사용
   join m.orders o // 경로 표현식 사용
where t.name = '팀A';// 경로 표현식 사용
```

#### 경로 표현식 용어 정리 
- 상태 필드: 단순히 값을 저장하기 위한 필드(필드 or 프로퍼티)
- 연관 필드: 연관 관계를 위한 필드 (임베디드 타입 포함)
- 단일 값 연관 필드: @ManyToOne, @OneToOne 대상 엔티티
- 컬렉션 값 연관 필드: @OneToMany, @ManyToMany 대상 컬렉션
```java
@Entity 
public class Member{
    @Id 
    @GeneratedValue 
    private Long id; 

    @Column(...)
    private String username; // 상태 필드 

    @ManyToOne(...)
    private Team team; // 연관 필드 (단일값)

    @OneToMany(...)
    private List<Order>orders; // 연관 필드(컬렉션값)
}
```
#### 경로 표현식과 특징 
- 상태 필드 경로
    - 경로 탐색의 끝이므로 더 탐색할 수 없다.

- 단일 값 연관 경로
    - 묵시적으로 내부 조인이 일어나고 계속 탐색할 수 있다.
    - 단일 값 연관 필드로 경로 탐색을 하면 SQL에서 내부 조인(묵시적 조인)이 일어난다.
    - 외부 조인은 명시적으로 JOIN키워드를 사용해야 한다.  
```java
select o.member from Order o
```
- 컬렉션 값 연관 경로 
    - t.members (컬렉션)까지는 경로탐색이 가능하지만 컬렉션에서 경로 탐색을 시작하는 것은 허락하지 않는다. 
    - 컬렉션에서 경로 탐색을 하고 싶으면 조인을 사용해야 한다. 

```java
select t.members from Team t; //성공
select t.members.username from Team t //실패 
```
    
```java
select m.username from Team t join t.members m 
```

#### 경로 탐색을 사용한 묵시적 조인시 주의사항 
- 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 힘들기 때문에 명시적 조인을 사용하는 것을 권장한다. 

### 10.2.9 서브쿼리 
서브쿼리는 WHERE, HAVING절에만 사용할 수 있고 SELECT, FROM절에는 사용할 수 없다. 
```java
select m from Member m 
where m.age > (select avg(m2.age) from Member m2) 
```
#### 서브쿼리 함수 
- EXISTS, ALL, ANY, SOME, IN 함수를 사용할 수 있다. 
### 10.2.10 조건식 
- 문자, 숫자, 날짜, Boolean, Enum, 엔티티 타입 

#### 연산자 우선 순위 
- 경로 탐색 연산 > 수학 연산 > 비교 연산 > 논리 연산 

#### 논리 연산과 비교식 
- AND, OR, NOR 

#### Between, IN, Like, NULL 비교 
- SQL에서 지원하는 문법과 동일하다. 

#### 컬렉션 식 
- 컬렉션식은 컬렉션에서만 사용하는 특별한 기능이다. 

```java
//JPQL 
select m from Member m
where m.orders is not empty

//SQL 
select m.* from Member m 
where 
    exists(
        select o.id 
        from Orders o
        where m.id=o.member_id
    )

```
#### 스칼라 식 
- 스칼라는 숫자, 문자, 날짜 case, 엔티티 타입 같은 가장 기본적인 타입을 말한다.
- 수학식(단항 연산자, 사칙연산)
- 문자 함수(SQL 함수) 
- 수학 함수(ABS, SQRT, MOD, SIZE, INDEX)
- 날짜 함수 

#### CASE식 
- 기본 CASE식 
- 심플 CASE식 
- COALESCE
- NULLIF

### 10.2.11 다형성 쿼리 
부모 엔티티를 조회하면 그 자식 엔티티도 함께 조회한다. 
```java
@Entity 
@Inheritance(strategy=InheritanceType.JOINED)
@DiscriminatorValue(name="DTYPE")
public abstract class Item{
    @Id
    @GeneratedValue
    @Column(name="ITEM_ID")
    private Long id; 

    private String name; 
    private int price; 
}
@Entity
@DiscriminatorValue("A")
public class Album extends Item{
    private String artist; 
    ...
}

@Entity 
@DiscriminatorValue("M")
public class Movie extends Item{
    private String director;
    private String actor;  
    ...
}

List resultList = em.createQuery("select i from Item i").getResultList();
```
#### Type 
- Type는 엔티티 상속 구조에서 조회 대상을 특정 자식 타입으로 한정할 때 주로 사용한다. 
```java
//JPQL
select i from Item i 
where type(i) IN (Book, Movie)

//SQL 
select i from Item i 
from i.DTYPE in ('B', 'M')
```
#### TREAT(JPA2.1) 
- Java의 타입 캐스팅과 비슷하다. 
- 상속 구조에서 부모 타입을 특정 장식 타입으로 다룰 때 사용한다. 
- JPA 표준은 FROM, WHERE절에서 사용할 수 있지만 하이버네이트는 SELECT절에서도 TREAT를 사용할 수 있다. 
```java 
//JPQL 
select i from Item i where treat(i as Book).author = 'kim' 
//SQL 
select i.* from Item i 
where i.DTYPE = 'B' 
and i.author = 'kim'
```
### 10.2.12 사용자 정의 함수(JPA2.1) 
```java
select function('group_concat', i.name) from Item i

public class MyH2Dialect extends H2Dialect{
    public MyH2Dialect() {
        registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicType.STRING));
    }
} 
```
```xml
<property name="hibernate.dialect" value="hello.MyH2Dialect">
```

### 10.2.13 기타 정리 
- enum은 = 비교 연산만 지원한다. 
- 임베디드 타입은 비교를 지원하지 않는다. 

#### EMPTY STRING
- JPA는 ''을 길이 0인 EMPTY STRING으로 정했지만 데이터베이스에 따라 ''를 NULL로 사용하는 데이터베이스도 있으므로 확인하고 사용해야 한다. 

#### NULL 정의 
- 조건을 만족하는 데이터가 없으면 NULL이다. 
- NULL은 알 수 없는 값이다. 
- NULL == NULL 은 알 수 없는 값이다. 
- NULL is NULL은 참이다. 

### 10.2.14 엔티티 직접 사용 
#### 기본키 값 
- 객체 인스턴스는 참조 값을 식별하고 테이블 로우는 기본 키 값으로 식별한다. 
- JPQL에서 엔티티 객체를 직접 사용하면 SQL에서는 해당 엔티티의 기본키 값을 사용한다. 
```java 
//JPQL
select count(m.id) from Member m 
select count(m) from Member m 
//SQL 
select count(m.id) as cnt 
from Member m 
```
```java
String qlString = "select m from Member m where m = :member";
List resultList = em.createQuery(qlString)
    .setParameter("member", member)
    .getResultList();

//SQL 
select m.* 
from Member m 
where m.id=?
```

#### 외래키 값 
```java
Team team = em.find(Team.class, 1L);

String query = "select m from Member m where m.team = :team";
List resultList = em.createQuery(query)
    .setParameter("team", team)
    .getResultList();
//SQL
select m.*
from Member m
where m.team_id=? (팀 파라미터의 ID 값)

String query = "select m from Member m where m.team.id = :teamId";
List resultList = em.createQuery(query)
    .setParameter("team", 1L)
    .getResultList();
```


### 10.2.15 Named 쿼리 : 정적 쿼리 
JPQL 쿼리는 크게 동적 쿼리와 정적 쿼리로 나눌 수 있다. 
#### 동적 쿼리
- em.createQuery("select ...") 처럼 JPQL을 문자로 완성해서 넘기는 것을 동적 쿼리라고 한다. 
#### 정적 쿼리 
- 미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용할 수 있다. 이것을 Named 쿼리라고 한다.
- Named쿼리는 애플리케이션 로딩 시점에 JPQL문법을 체크하고 미리 파싱해 두기 때문에 오류를 빨리 확인 가능하고 사용하는 시점에 파싱된 결과를 재사용해서 성능상의 이점이 있다. 

#### Named쿼리를 어노테이션에 정의 
```java
@Entity 
@NamedQuery(name="Member.findByUsername", 
            query = "select m from Member m where m.username = :username")
public class Member {
    ...
}

List<Member> resultList = em.createQuery("Member.findByUsername", Member.class)
    .setParameter("username", "회원1")
    .getResultList();

```
- 하나의 엔티티에 2개 이상의 Named 쿼리를 정의하려면 @NamedQueries 어노테이션을 사용하면 된다. 

```java
@Entity 
@NamedQueries({
    @NameQuery(
        name="Member.findByUsername", 
        query = "select m from Member m where m.username = :username"),
    @NameQuery(
        name="Member.findByUsername2", 
        query = "select m from Member m where m.username = :username") })
public class Member {
    ...
}
```
#### Named쿼리를 XML에 정의 
- JPA에 어노테이션으로 작성할 수 있는 것은 XML로도 작성이 가능하다. 
- 어노테이션을 사용하는 것이 직관적이고 편리하지만 Named쿼리를 작성할 때는 XML을 사용하는 것이 편리하다. 

#### 환경에 따른 설정 
- XML과 어노테이션에 같은 설정이 있으면 XML이 우선권을 가진다. 