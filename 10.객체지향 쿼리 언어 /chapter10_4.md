# 10장 객제지향 쿼리 언어 
## 10.4 QueryDSL 
Criteria는 문자가 아니라 코드로 JPQL을 작성하므로 문법 오류를 컴파일 단계에서 잡을 수 있고 IDE의 자동완성 기능의 도움을 받을 수 있는등 여러가지 장점이 있지만 너무 복잡하고 어렵다. 쿼리를 문자가 아닌 코드로 작성해도 쉽고 간결하고 그 모양도 쿼리와 비슷하게 개발할 수 있는 프로젝트가 QueryDSL이다. 
### 10.4.1 QueryDSL 설정 
#### 필요라이브러리 
#### 환경설정 
- QueryDSL을 사용하려면 Criteria의 메타 모델 처럼 엔티티를 기반으로 쿼리 타입이라는 쿼리용 클래스를 생성해야 한다. 
#### 10.4.2 시작 
```java
public void queryDSL() {
	EntityManager em = emf.createEntityManager();
		
	JPAQuery query = new JPAQuery(em);
	QMember qMember = new QMember("m"); //생성되는 JPQL의 별칭이 m

	List<Member> members = query.from(qMember)
								.where(qMember.name.eq("회원"))
								.orderBy(qMember.name.desc())
								.list(qMember); 
}
/* JPQL:
select m from Member m
 where m.name = ?1
 order by m.name desc
*/
```
- QueryDSL을 사용하려면 우선 JPAQuery객체를 생성해야 하는데 이때 엔티티 매니저를 생성자에 넘겨준다. 
- 사용할 쿼리 타입(Q)을 생성하는데 생성자에는 별칭을 주면 된다. 

#### 기본 Q생성 
- 쿼리 타입은 사용하기 편리하도록 기본 인스턴스를 보관하고 있지만 같은 엔티티를 조인하거나 같은 엔티티를 서브쿼리에 사용하면 같은 별칭이 되므로 이때는 별칭을 직접 지정해서 사용해야 한다. 
```java
QMember qMember = new QMember("m"); //직접 지정
QMember qMember = QMember.member;   //기본 인스턴스 사용
import static jpabook.jpashow.doamin.QMember.member; //기본 인스턴스
```
- 쿼리 타입의 기본 인스턴슬를 사용하면 아래와 같이 import static를 사용하면 코드를 더욱 간결하게 작성이 가능하다. 
```java
public void basic() {
	EntityManager em = emf.createEntityManager();
		
	JPAQuery query = new JPAQuery(em);
	List<Member> members = query.from(member)
								.where(member.name.eq("회원"))
								.orderBy(member.name.desc())
								.list(member); 
	}
```
### 10.4.3 검색 조건 쿼리 
```java
//QueryDSL
JPAQuery query = new JPAQuery(em);
QItem item = QItem.item;
List<Item> list = query.from(item)
						.where(item.name.eq("좋은상품").and(item.price.gt(20000)))
						.list(item);

//JPQL
select item
 from Item item
where item.name = ?1 and item.price >?2
```
- QueryDSL의 where절에는 and나 or을 사용할 수 있다. 아래와 같이 사용하면 and 연산이 된다. 

```java
.where(item.name.eq("상품"), item.price.gt(10000))
```
- 쿼리 타입의 필드는 필요한 대부분의 메소드를 명시적으로 제공한다. 

### 10.4.4 결과 조회 
- 쿼리 작성이 끝나고 결과 조회 메소드를 호출하면 실제 데이터베이스를 조회한다. 
- uniqueResult()나 list()를 사용하고 파라미터로 프로젝션 대상을 넘겨준다. 
- uniqueResult() 
	- 조회 결과가 한 건일 때 사용한다.
	- 조회 결과가 없으면 null을 반환한다. 
	- 하나 이상이면 NonUniqueResultException 예외가 발생한다. 
- singleResult()
	- uniqueResult()와 같지만 결과가 하나 이상이면 **처음 데이터를 반환한다**. 
- list()
	- 결과가 하나 이상일 때 사용하고, 결과가 없으면 빈 컬렉션을 반환한다. 

### 10.4.5 페이징과 정렬 
- 정렬은 orderBy를 사용하는데 쿼리 타입이 제공하는 asc(), desc()를 사용한다. 
- 페이징은 offset과 limit를 적절히 조합해서 사용하면 된다. 
```java
QItem item = QItem.item;

query.from(item)
		.where(item.price.gt(20000))
		.orderBy(item.price.desc(), item.stockQuantity.asc())
		.offset(10).limit(20)
		.list(item);

QueryModifiers queryModifiers = new QueryModifiers(20L, 10L);
List<Item> list = query.from(item)
		.restrict(queryModifiers)
		.list(item);
```
- 실제 페이징 처리를 하려면 검색된 전체 데이터 수를 알아야 한다. 
```java
SearchResults<Item> result =
	query.from(item) 
		 .where(item.price.gt(20000))
		 .offset(10).limit(20) 
		 .listResults(item)
long total = result.getTotal(); // 검색 전체 데이터 수 
long limit = result.getLimit();  
long offest = result.getOffset();
List<Item>results = result.getResults(); 

```
### 10.4.6 그룹 
- 그룹은 groupBy를 사용하고 그룹된 결과를 제한하기 위해서는 having을 사용한다. 
```java
query.from(item)
	 .groupBy(item.price)
	 .having(item.price.gt(1000))
	 .list(item);
```
### 10.4.7 조인 
- 조인은 innerJoin, leftJoin, rightJoin, fullJoin을 사용할 수 있고 JPQL의 on 성능과 최적화를 위한 fetch조인도 사용할 수 있다. 
- 첫번째 파라미터에 조인 대상을 지정, 두 번째 파라미터에 별칭으로 사용할 쿼리 타입을 지정한다. 

```java
//기본
QOrder order = QOrder.order;
QMember member = QMember.member;
QOrderItem orderItem = QOrderItem.orderItem;

query.from(order)
		.join(order.member, member)
		.leftJoin(order.orderItems, orderItem)
		.list(order);
// On 사용
query.from(order)
		.leftJoin(order.orderItems, orderItem)
		.on(orderItem.count.gt(2))
		.list(order);
// fetch 사용 
query.from(order)
		.innerJoin(order.member, member).fetch()
		.leftJoin(order.orderItems, orderItem).fetch()
		.list(order);
// 세타 조인 
query.from(order, member)
		.where(order.member.eq(member))
		.list(order);
```
### 10.4.8 서브쿼리 
- 서브쿼리는 JPASubQuery를 생성해서 사용한다. 
- 서브 쿼리의 결과가 하나면 unique(), 여러 건이면 list()를 사용할 수 있다. 
```java
//서브쿼리 - unique
QItem item = QItem.item;
QItem itemSub = new QItem("itemSub");

query.from(item)
	 .where(item.price.eq(new JPASubQuery().from(itemSub)
						.unique(itemSub.price.max())))
	 .list(item);

//서브쿼리 - list
query.from(item)
	 .where(item.in(new JPASubQuery().from(itemSub)
						.where(item.name.eq(itemSub.name))
						.list(itemSub)
		))
		.list(item);

```
### 10.4.9 프로젝션과 결과 반환 
- select 절에 조회 대상을 지정하는 것을 프로젝션이라고 한다. 
#### 프로젝션 대상이 하나 

```java
QItem item = QItem.item;
List<String> result = query.from(item).list(item.name);

for (String name : result) {
		System.out.println("name = " + name);
}

```
#### 여러 컬럼 반환과 튜플 
- 프로젝션을 대상으로 여러 필드를 선택하면 QueryDSL은 기본은로 Tuple이라는 Map과 비슷한 내부 타입을 사용한다. 
- 조회결과는 tuple.get()메소드에 조회한 타입을 지정하면 된다. 
```java
QItem item = QItem.item;
List<Tuple> result = query.from(item).list(item.name, item.price);
//List<Tuple> result = query.from(item).list(new QTuple(item.name, item.price));

for (Tuple tuple : result){
    System.out.println("name = " + tuple.get(member.name));
    System.out.println("age = " + tuple.get(member.age));
}
```
#### 빈 생성 
- 쿼리 결과를 엔티티가 아닌 특정 객체로 받고 싶으면 빈 생성 기능을 사용하면 되고 QueryDSQL은 객체를 생성하는 다양한 방법을 제공한다. 
- Projections.bean()메소드는 Setter를 사용해서 값을 채운다. 
- 쿼리 결과는 name인데 ItemDTO는 username 프로퍼티를 가지고 있는데 쿼리 결과와 매핑할 프로퍼티 타입이 다르면 as를 사용해서 별칭을 주면 된다. 

```java
//엔티티가 아닌 DTO 생성
public class ItemDTO {
	private String username;
	private int pirce;

	public ItemDTO() {}
	public ItemDTO(String username, int price) {
		this.username = username;
		this.price = price;
	}

	//Getter, Setter 
	...
}

// 프로퍼티 접근 
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
		Projections.bean(ItemDTO.class, item.name.as("username"), item.price));

```

- Projections.fields() 메소드를 사용하면 필드에 직접 접근해서 값을 채워 준다. 필드를 private 로 설정해도 동작한다.  
```java
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
		Projections.fields(ItemDTO.class, item.name.as("username"), item.price));

```
- Projections.constrouctor() 메소드는 생성자를 지원한다. 지정한 프로젝션과 파라미터 순서가 같은 생성자가 필요하다. 

```java
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
		Projections.constructor(ItemDTO.class, item.name.as("username"), item.price));
```
#### DISTINCT
```java
query.distinct().from(item)...
```
### 10.4.10 수정, 삭제, 배치 쿼리 
- JPQL 배치 쿼리와 같이 영속성 컨텍스트를 무시하고 데이터베이스를 직접 쿼리한다는 점에 유의하도록 한다. 
```java
// 수정 배치 쿼리 
QItem item = QItem.item;

JPAUpdateClause updateClause = new JPAUpdateClause(em, item);
long count = updateClause.where(item.name.eq("jpa 책"))
		.set(imte.price, item.price.add(100))
		.execute();

// 수정 삭제 쿼리 
QItem item = QItem.item;

JPADeleteClause deleteClause= new JPADeleteClause(em, item);
long count = deleteClause.where(item.name.eq("jpa 책"))
		.execute();
```
### 10.4.11 동적 쿼리 
- BooleanBuilder를 사용하면 특정 조건에 따른 동적 쿼리를 편리하게 사용 가능하다. 
```java
SearchParam param = new  SearchParam();
param.setName("시골개발자");
param.setPrice(10000);

QItem item = QItem.item;

BooleanBuilder builder = new BooleanBuilder();
if (StringUtils.hasText(param.getName())) {
		builder.and(item.name.contains(param.getName()));
}
if (param.getPrice() != null) {
		builder.and(item.price.gt(param.getPrice()));
}

List<Item> result = query.from(item)
		.where(builder)
		.list(item);
```

### 10.4.12 메소드 위임
- 메소드 위임 기능을 사용하면 쿼리 타입에 검색 조건을 직접 정의할 수 있다. 
```java
public class ItemExpression {
	@QueryDelegate
	public static BooleanExpression isExpensive(QItem item, Integer price) {
			return item.price.gt(price);
	}
}

public class QItem extends EntityPathBase<Item> {
		...
		public BooleanExpression isExpensive(Integer price) {
        return ItemExpression.isExpensive(this, price);
    }
}

query.from(item).where(item.isExpensive(30000)).list(item);
```