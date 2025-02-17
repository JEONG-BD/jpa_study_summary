# 10장 객제지향 쿼리 언어 
## 10.5 네이티브 SQL  
JPQL은 표준 SQL이 지원하는 대부분의 문법과 SQL함수들을 지원하지만 특정 데이터베이스에 종속적인 기능은 지원하지 않는다. 때로는 특정 데이터베이스에 종속적인 기능이 필요하고, JPA는 특정 데이터베이스에 종속적인 기능을 사용할 수 있는 다양한 방법을 열어 두었다.  

 - 특정 데이터베이스만 지원하는 함수
	- JPQL에서 네이티브 SQL함수를 호출 가능
	- 하이버네이트는 데이터베이스 방언에 각 데이터베이스에 종속적인 함수들을 정의해 두었으며 직접 호출할 함수를 정의 가능
 - 특정 데이터베이스만 지원하는 SQL 쿼리 힌트
	- 하이버네이트를 포함한 몇몇 구현체들이 지원 
 - 인라인 뷰, UNION, INTERSECT
	- 하이버네이트는 지원하지 않지만 몇몇 JPA 구현체들은 지원
 - 스토어드 프로시저
	- JPQL에서는 스토어드 프로시저를 호출 가능 
 - 특정 데이터베이스만 지원하는 문법 
	- 특정 데이터베이스에 너무 종속적인 SQL문법은 지원 불가능하기 때문에 이때는 네이티브 SQL 사용 

- 다양한 이유로 JPQL을 사용할 수 없을 때 SQL을 직접 사용할 수 있는 기능을 제공한다. 
- JPA가 지원하는 네이티브 SQL은 엔티티를 조회가능하고 JPA가 지원하는 영속성 컨텍스트의 기능을 그대로 사용가능 하지만 JDBC API는 단순히 데이터의 나열을 조회할 뿐이다. 
### 10.5.1 네이티브 SQL 사용 
- 네이티브 쿼리 API는 세 가지가 있다. 
```java

// 결과 타입 정의 
public Query createNativeQuery(String sqlString, Class resultClass);

// 결과 타입을 정의할 수 없을 경우 
public Query createNativeQuery(String sqlString)

// 결과 매핑 사용 

public Query createNativeQuery(String sqlString, String resultSetMapping )
```
#### 엔티티 조회 
```java
String sql = 
	"SELECT ID, AGE, NAME, TEAM_ID " +
	"FROM MEMBER WHERE AGE > ? ";

Query nativeQuery = em.createNativeQuery(sql, Member.class).setParameter(1, 20); 

List<Member> resultList = nativeQuery.getResultList();
```

- 네이티브 SQL로 SQL만 사용할 뿐이지 나머지는 JPQL을 사용할 때와 같아. 조회한 엔티티도 영속성 컨텍스트에서 관리된다. 
#### 값 조회 
```java
String sql = 
		"SELECT ID, AGE, NAME, TEAM_ID " +
		"FROM MEMBER WHERE AGE > ? ";

Query nativeQuery = em.createNativeQuery(sql)
					.setParameter(1, 10);

List<Object[]> resultList = nativeQuery.getResultList();
for (Object[] row : resultList) {
	System.out.println("id = " + row[0]);
	System.out.println("age = " + row[1]);
	System.out.println("name = " + row[2]);
	System.out.println("team_id = " + row[3]);
}

```
- 여기서는 스칼라 값들은 조회했을 뿐이므로 결과를 영속성 컨텍스트에서 관리하지 않는다. 
#### 결과 매핑 사용 
- 엔티티와 스칼라 값을 함께 조회하는 것처럼 매핑이 복잡해지면 @SqlResultSetMapping을 정의해서 결과 매핑을 사용해야 한다. 
```java
String sql = 
	"SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT " +
	"FROM MEMBER M " +
	"LEFT JOIN " +
	"  (SELECT IM.ID, COUNT(*) AS ORDER_COUNT " +
	"   FROM ORDERS O, MEMBER IM " +
	"  WHERE O.MEMBER_ID = IM.ID) I " +
	"ON M.ID = I.ID";

Query nativQuery = em.createNativeQuery(sql, "memberWithOrderCount");
List<Ojbect[]> resultList = nativeQuery.getResultList();

for (Ojbect[] row : resultList) {
	Member member = (Member) row[0];
	BigInteger orderCount = (BigInteger)row[1];

	System.out.println("member = " + member);
	System.out.println("orderCount = " + orderCount);
}

@Entity
//1. Entity에 바로 매핑
@SqlResultSetMapping(name = "memberWithOrderCount",
	entities = {@EntityResult(entityClass = Member.class)},
	columns = {@ColumnResult(name = "ORDER_COUNT")}
)

//2. Entity에 매핑할 컬럼 지정
@SqlResultSetMapping(name = "memberWithOrderCount",
	entities = {@EntityResult(entityClass = Member.class, fields = {
				@FieldResult (name="id", column="order_id"),
				@FieldResult (name="quantity", column="order_quantity"),
				...
		})},
		columns = {@ColumnResult(name = "ORDER_COUNT")}
)
public class Member {...}
```
#### 결과 매핑 어노테이션 
- @SqlResultSetMapping
- @EntityResult 
- @FieldResult 
- @ColumnResult 

### 10.5.2 Named 네이티브 SQL 
- JPQL처럼 네이티브 SQL Named 네이티브 SQL을 사용해서 정적 SQL을 작성 가능하다. 
```java
//Entity Named 네이트브 SQL 설정
@Entity
@NamedNativeQuery(
	name = "Member.memberSQL",
	query = "SELECT ID, AGE, NAME, TEAM_ID " +
			  "FROM MEMBER WHERE AGE > ?",
	resultClass = Member.class
)
public class Member {...}

//사용
TypeQuery<Member> nativeQuery = 
	em.createNamedQuery("Member.memberSQL", Member.class)
	.setParameter(1, 20);

// Named 네이티브 SQL 결과 매핑
@Entity
@SqlResultSetMapping(name = "memberWithOrderCount",
	entities = {@EntityResult(entityClass = Member.class)},
	columns = {@ColumnResult(name = "ORDER_COUNT")}
)
@NamedNativeQuery(
	name = "Member.memberWithOrderCount",
	query = "SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT " +
			"FROM MEMBER M " +
			"LEFT JOIN " +
			"     (SELECT IM.ID, COUNT(*) AS ORDER_COUNT " +
			"     FROM ORDERS O, MEMBER IM " +
			"     WHERE O.MEMBER_ID = IM.ID) I " +
			"ON M.ID = I.ID",   
	resultSetMapping = "memberWithOrderCount"
)
public class Member {...}

//실행
List<Object[]> resultList = em.createNamedQuery("Member.memberWithOrderCount")
                .getResultList();

```
#### @NamedNativeQuery
|속성|기능|
|---|---|
|name|네임드 쿼리 이름|
|query|SQL 쿼리|
|hints|벤더 종속적인 힌트|
|resultClass|결과 클래스| 
|resultSetMapping|결과 매핑 사용|

### 10.5.3 네이티브 SQL XML에 정의
```xml
<entity-mappings ..>
		<named-native_query name="Member.memberWithOrderCountXml">
		</named-native_query>

		<sql-result-set-mapping name="memberWithOrderCountResultMap">
			<entity-result entity-class="jpabook.domain.Member" />
			<column-result name="ORDER_COUNT" />
		</sql-result-set-mapping>
```
- XML에 정의할 때는 순서를 지켜야 한다. 
- <named-native_query>를 먼저 정의하고 <sql-result-set-mapping>을 정의해야 한다. 
- 네이티브 SQL은 보통 JPQL로 작성하기 어려운 복잡한 SQL 쿼리를 작성하거나 SQL을 최적화해서 데이터베이스 성능 향상을 할때 사용한다. 
- XML을 사용하는 것이 편리하다. 
### 10.5.4 네이티브 SQL 정리 
- 네이티브 SQL은 관리하기 쉽지 않고 자주 사용하면 특정 데이터베이스에 종속적인 쿼리가 증가해서 이식성이 떨어진다. 
- 될 수 있으면 JPQL을 사용하고 기능이 부족하면 하이버네이트 같은 JPA구현체가 제공하는 기능을 사용하고 마지막으로 네이티브 SQL을 사용하도록 한다. 
### 10.5.5 스토어드 프로시저 
```sql
DELIMITER //

CREATE PROCEDURE proc_multiply (INOUT inParam INT, INTOUT outParam INT)
BEGIN
	SET outParam = inParam * 2;
END //

```
```java
 StoredProcedureQuery spq = em.createStoredProcedureQuery("proc_multiply");
 spq.registerStoredProcedureParameter(1, Integer.class, ParameterMode.IN);
 spq.registerStoredProcedureParameter(2, Integer.class, ParameterMode.OUT);

 spq.setParameter(1, 100);
 spq.execute();
 Integer outputParameterValue = (Integer) spq.getOutputParameterValue(2);
 System.out.println(outputParameterValue); // 결과: 200

```
- 스토어드 프로시저를 사용하려면 m.createStoredProcedureQuery()에 사용할 스토어드 프로시저 이름을 입력하면 된다. 
- registerStoredProcedureParameter()를 사용해서 프로시저에서 사용할 파라미터 순서, 타입, 모드순으로 정의하면 된다. 

```java
public enum ParamterMode {
		IN,         //INPUT 파라미터
		INOUT,      //INPUT, OUTPUT 파라미터 
		OUT,        //OUTPUT 파라미터 
		REF_CURSOR  //CURSOR 파라미터
}
```
#### Named 스토어드 프로시저 사용 
- 스토어드 프로시저 쿼리에 이름을 부여해서 사용하는 것을 Named 스토어드 프로시저라고 한다. 
```java
@NamedStoredProcedureQuery(
		name = "multiply",
		procedureName = "proc_multiply",
		paramters = {
				@StoredProcedureParamter(name = "inParam", mode = 
						ParamterMode.IN, type = Integer.class),
				@StoredProcedureParamter(name = "outParam", mode = 
						ParamterMode.OUT, type = Integer.class)
		}
)
@Entity
public class Member {...}

```