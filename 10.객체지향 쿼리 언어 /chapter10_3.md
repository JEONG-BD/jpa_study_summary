# 10장 객제지향 쿼리 언어 
## 10.3 Criteria 
Criteria 쿼리는 JPQL를 Java 코드로 작성하도록 도와주는 빌더 클래스 API다. Criteria를 사용하면 문자가 아닌 코드로 JPQL을 작성하므로 문법 오류를 컴파일 단계에서 잡을 수 있고 문자 기반의 JPQL보다 동적 쿼리를 안전하게 생성할 수 있다는 장점이 있지만 코드가 복잡해서 직관적으로 이해가 떨어진다. 

### 10.3.1 Criteria 기초 
- Criteria API는 javax.persistence.criteria 패키지에 있다. 
```java 
CriteriaBuilder cb = em.getCriteriaBuilder();

CriteriaQuery<Member> cq = cb.createQuery(Member.class); 

Root<Member> m = cq.from(Member.class); 
cq.select(m);

TypeQuery<Member> query = em.createQuery(cq);
List<Member> members = query.getResultList();

RootM<Member> m = cq.from(Member.class);

// 검색 조건 정의 
Predicate usernameEqual = cb.equal(m.get("username"), "회원1");

// 정렬 조건 정의 
javax.persistence.criteria.Order ageDesc = cb.desc(m.get("age"));


cq.select(m)
    .where(usernameEqual)
    .orderBy(ageDesc);
```

### 10.3.2 Criteria 쿼리 생성 
- Criteria를 사용하려면 CriteriaBuilder.createQuery()로 Criteria쿼리를 생성하면 된다. 
- Criteria 쿼리를 생성할 때 파라미터로 쿼리 결과에 대한 반환 타입을 지정할 수 있다. 
```java
CriteriaBuilder cb = em.getCriteriaBuilder(); 

CriteriaQuery<Member> cq = cb.createQuery(Member.class); 
...
```
- 반환 타입을 지정할 수 없거나 반환 타입이 둘 이상이면 Object로 반환받으면 된다. 
```java
CriteriaBuilder cb = em.getCriteriaBuilder(); 

// return Object
CriteriaQuery<Object> cq = cb.createQuery(); 
List<Object>resultList = em.createQuery(cq).getResultList();
... 

// return Object[]
CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class); 
List<Object>resultList = em.createQuery(cq).getResultList();

// return Tuple 
CriteriaQuery<Tuple> cq = cb.createQuery(Object[].class); 
TypedQuery<Tuple> query = em.createQuery(cq)
```
### 10.3.3 조회 
- select에 조회 대상을 하나만 지정하려면 select()를 사용하고 여러 건을 지정하려면 multiselect() 또는 cb.array()를 사용한다. 
```java 
// JPQL select m 
cq.select(m) 

// JPQL select m.username, m.age 
cq.multiselect(m.get("username"), m.get("age"));  

CriteriaBuilder cb = em.createCriteriaBuilder();
cq.select(cb.array(m.get("username"), m.get("age"))); 

```
#### DISTINCT 
- distinct는 select, multiselect다음에 distinct(true)를 사용하면 된다. 
```java
...
cq.select(cb.array(m.get("username"), m.get("age"))).distinct(true); 
...
```
#### New, construct()
- JPQL에서 select new 생성자() 생성자 구문을 Criteria 에서는 cb.construct(클래스 타입)로 사용한다. 
```java
//JPQL select new jpabook.domain.MemberDTO(m.username, m.age) from Member m;

CriteriaQuery<MemberDTO> cq = cb.createQuery(MemberDTO.class); 
Root<Member> m = cq.from(Member.class); 
cq.select(cb.construct(MemberDTO.class, m.get("username"). m.get("age")));
```
#### 튜플 
- Criteria는 Map와 비슷한 튜플이라는 특별한 반환 객체를 제공한다. 
- 튜플을 사용하려면 cb.createTupleQuery()또는 cb.createQuery(Tuple.class)로 Criteria를 생성한다. 
- 튜플은 튜플의 검색 키로 사용할 튜플 전용 별칭을 필수로 할당해야 한다. 
```java
//JPQL: select m.username, m.age from Member m

CriteriaBuilder cb = em.getCriteriaBuilder();

CriteriaQuery<Tuple> cq = cb.createTupleQuery();

Root<Member> m = cq.from(Member.class);
cq.multiselect(
		m.get("username").alias("username"), //튜플에서 사용할 별칭 
		m.get("age").alias("age")
);

TypedQuery<Tuple> query = em.createQuery(cq);
List<Tuple> resultList = query.getResultList();
for (Tuple tuple : resultList) {
		//튜플 별칭 조회 
		String username = tuple.get("username", String.class);
		Integer age = tuple.get("age", Integer.class);
}
```

### 10.3.4 집합 
```java
/*
JPQL
select m.team.name, max(m.age), min(m.age)
  from Member m
  group by m.team.name
*/

CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class);
Root<Member> m = cq.from(Member.class);

Expression maxAge = cb.max(m.<Integer>get("age"));
Expression minAge = cb.min(m.<Integer>get("age"));

cq.multiselect(m.get("team").get("name"), maxAge, minAge);
cq.groupBy(m.get("team").get("name"));
TypedQuery<Object[]> query = em.createQuery(cq);
List<Object[]> resultList = query.getResultList();
```
#### HAVING 
```java
cq.multiselect(m.get("team").get("name"), maxAge, minAge)
.groupBy(m.get("team").get("name"))
.having(cb.gt(minAge, 10));

```

### 10.3.5 정렬 
- 정렬 조건도 Criteria 빌더를 통해서 생성한다. 

```java
//JPQL: order by m.age desc
cq.select(m)
	.where(ageGt)
	.orderBy(cb.desc(m.get("age"))); 
```

### 10.3.6 조인 
- 조인은 join()메소드와 JoinType클래스를 사용한다. 
```java
public enum JoinType {
		INNER, //내부조인
		LEFT, //왼쪽 외부 조인
		RIGHT //오른쪽 외부 조인, JPA 구현체나 데이터베이스에 따라 지원 불가 
}

/* 
JPQL
select m, t from Member m
 inner join m.team t
 where t.name = '팀A'
*/
Root<Member> m = cq.from(Member.class);
Join<Member, Team> t = m.join("team", JoinType.INNER); //내부 조인

cq.multiselect(m, t)
	.where(cb.equal(t.get("name"), "팀A"));


1. m.join("team") //내부 조인 (JoinType.INNER 생략 가능)
2. m.join("team", JoinType.LEFT) //LEFT 조인
3. m.fetch("team", JoinType.LEFT) //FETCH JOIN
```
### 10.3.7 서브쿼리 
```java
/* JPQL:
select m from Member m
 where m.age >=
	   (select AVG(m2.age) from Member m2)
*/
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> mainQuery = cb.createQuery(Member.class);

//서브 쿼리 생성
Subquery<Double> subQuery = mainQuery.subquery(Double.class);
Root<Member> m2 = subQuery.from(Member.class);
subQuery.select(cb.avg(m2.<Integer>get("age")));

//메인 쿼리 생성
Root<Member> m = mainQuery.from(Member.class);
mainQuery.select(m)
         .where(cb.ge(m.<Integer>get("age"), subQuery))
```
### 10.3.8 IN식

```java
/* JPQL:
select m from Member m
 where m.username in ("회원1", "회원2")
*/
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> cq = cb.createQuery(Member.class);
Root<Member> m = cq.from(Member.class);

cq.select(m)
  .where(cb.in(m.get("username"))
  .value("회원1")
  .value("회원2"));

```
### 10.3.9 CASE식 
```java
/* JPQL
select m.username
       case when m.age>=60 then 600
	        when m.age<=15 then 500
			else 1000
			end
 from Member m
*/
Root<Member> m = cq.from(Member.class);

cq.multiselect(
	m.get("username"),
	cb.selectCase()
	.when(cb.ge(m.<Integer>get("age"), 60), 600)
	.when(cb.le(m.<Integer>get("age"), 15), 500)
	.otherwise(1000)
);
```

### 10.3.10 파라미터 정의
- JPQL에서 :PAARAM1 처럼 파라미터를 정의했듯이 Criteria도 파라미터를 정의할 수 있다. 
```java
/*
JPQL 
select m 
  from Member m 
 where m.username 
*/
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> cq = cb.createQuery(Member.class);

Root<Member> m = cq.from(Member.class);


cq.select(m)
  .where(cb.equal(m.get("username"), 
         cb.parameter(String.class, "usernameParam"))); //파라미터 타입 정의 

List<Member> resultList = em.createQuery(cq)
		.setParameter("usernameParam", "회원1") //파라미터 값 바인딩 
		.getResultList();

```
### 10.3.11 네이티브 함수 호출 
- 네이티브 SQL 함수를 호출하려면 cb.function(...)를 사용하면 된다. 
```java
Root<Member> m = cq.from(Member.class);
Expression<Long> function = cb.function("SUM", Long.class, m.get("age"));
cq.select(function);
```
- 하이버네이트 구현체에서는 방언에 사용자 정의 SQL함수를 등록해야 호출할 수 있다. 

### 10.3.12 동적 쿼리 
- 다양한 검색 조건에 따라 실행 시점에 쿼리를 생성하는 동적쿼리는 문자 기반인 JPQL보다 코드 기반인 Criteria로 작성하는 것이 훨씬 더 편리하다. 
```java
//JPQL 동적 쿼리 
Integer age = 10;
String username = null;
String teamName = "팀A";

//JPQL 동적 쿼리 생성
StringBuilder jpql = new StringBuilder("select m from Member m join m.team t ");
List<String> criteria = new ArrayList<String>();

if (age != null) criteria.add(" m.age = :age ");
if (username != null) criteria.add(" m.username = :username ");
if (teamName != null) criteria.add(" t.name = :teamName ");

if (criteria.size() > 0 ) jpql.append(" where ");

for (int i = 0; i< criteria.size(); i++) {
		if(i > 0) jpql.append(" and ");
		jpql.append(criteria.get(i));
}

TypeQuery<Member> query = em.createQuery(jpql.toString(), Member.class);
if (age != null) query.setParameter("age", age);
if (username != null) query.setParameter("username", username);
if (teamName != null) query.setParameter("teamName", teamName);

List<Member> resultList = query.getResultList();

//Criteria 동적 쿼리
Integer age = 10;
String username = null;
String teamName = "팀A";

CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> cq = cb.createQuery(Member.class);

Root<Member> m = cq.from(Member.class);
Join(Member, Team) t = m.join("team");

List<Predicate> criteria = new ArrayList<Predicate>();
if (age != null) criteria.add(cb.equal(m.<Integer>get("age"),
	 cb.parameter(Integer.class, "age")));
if (username != null) criteria.add(cb.equal(m.get("username"),
	 cb.parameter(String.class, "username")));
if (teamName != null) criteria.add(cb.equal(t.get("name"),
	 cb.parameter(String.class, "teamName")));

cq.where(cb.and(criteria.toArray(new Predicate[0])));

TypedQuery<Member> query = em.createQuery(cq);
if (age != null) query.setParameter("age", age);
if (username != null) query.setParameter("username", username);
if (teamName != null) query.setParameter("teamName", teamName);

List<Member> resultList = query.getResultList();r
```
### 10.3.13 함수 정리 
- JPQL에사 사용하는 함수는 대부분 CriteriaBuilder에 정의되어 있다. 
- 조건함수 
- 스칼라와 기타함수 
- 집합 함수 
- 분기 함수 

### 10.3.14 Criteria 메타 모델 API 
- Criteria는 코드 기반이므로 컴파일 시점에 오류를 발견 가능하다. 
- m.get("age")에서 "age"대신에 "ageaaa"를 입력할 경우에는 컴파일 시점에 발견이 불가능하다. 
- 이러한 부분까지 코드로 작성하려면 메타 모델 API를 사용하면 된다. 
```java
// 메타 모델 API 전 
cq.select(m)
  .where(cb.gt(m.<Integer>get("username"), 20))
  .orderBy(cb.desc(m.get("age")));

// 메타 모델 API 후 
cq.select(m)
  .where(cb.gt(m.get(Member_.age), 20))
  .orderBy(cb.desc(m.get(Member_.age)));

```
- 메타 모델 API를 쓰기 위해서는 Member_ 클래스가  필요하다. 
- Member_ 메타 모델 클래스는 Member 엔티티를 기반으로 만들어야 하며, 개발자가 직접 작성하지 않는다. 
- 코드 생성시가 엔티티 클래스를 기반으로 메타 모델 클래스를 만들어 준다. 
- Criteria를 자주 사용한다면 메타모델 클래스를 적용하는 것을 권장한다. 

