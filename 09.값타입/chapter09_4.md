# 9장 값타입 

### 9.4 값 타입의 비교 

```java
int a = 10;
int b = 10; 

Address a = new Address("서울시", "종로구", "1번지");
Address b = new Address("서울시", "종로구", "1번지");
```

- 자바가 제공하는 객체 비교는 2가지다. 
#### 동일성(Identity) 비교
- 인스턴스의 참조 값을 비교한다 == 사용  
#### 동등성(Equlvalence) 비교
- 인스턴스의 값을 비교한다. equals()사용 
- Address 값 타입을 a == b로 동일성을 비교하면 둘은 서로 다른 인스턴스 이므로 결과는 거짓이다. 
- 값 타입은 비록 인스턴스가 달라도 그 안에 값이 같이 같으면 같은 것으로 봐야 한다. 
- 값 타이블 비교할 때는 a.equals(b)를 사용해서 동등성 비교를 해야 하며 Address의 equals() 메소드를 재정의 해야한다. 


### 9.5 값 타입 컬렉션 
- 값 타입을 하나 이상 저장하려면 컬렉션에 보관하고 @ElementCollection, @ColletionTable어노테이션을 저장하면 된다. 

```java
@Entity 
public class Member{

    @Id 
    @GeneratedValue 
    private Long id;

    @Embedded
    private Address homeAddress; 

    @ElementCollection
    @CollectionTable(name = "FAVORITE_FOODS", 
                        joinColumns = @JoinColumn(name="MEMBER_ID"))
    @Column(name="FOOD_NAME")
    private Set<String> favoriteFoods = new HashSet<String>();

    @ElementCollection
    @CollectionTable(name="ADDRESS", 
                        joinColumns = @JoinColumn(name = "MEMBER_ID"))
    private List<Address> addressHistory = new ArraryList<Address> ();
}

@Embeddable 
public class Address{
    @Column
    private String city; 
    private String street; 
    private String zipcode; 
}
```

### 9.5.1 값타입 컬렉션 사용 
```java
Member member = new Member(); 

member.setHomeAddress(new Address("통영", "몽돌해수욕장", "660-123")); 

//기본값 타입 컬렉션 
member.getFavoriteFoods().add("짬뽕");
member.getFavoriteFoods().add("짜장");
member.getFavoriteFoods().add("탕수육");

member.getAddressHistory().add(new Address("서울", "강남", "123-111"))
member.getAddressHistory().add(new Address("서울", "강북", "123-222"))

em.persist();
```
- 값 타입 컬렉션은 영속성 전이(Cascade) + 고아 객체 기능을 필수로 가진다고 볼 수 있다. 
- 값 타입 컬렉션을 조회할 때 페치 전략을 선택할 수 있는데 LAZY가 기본이다. 

```java
Member member = em.find(Member.class, 1L);

//1번 임베디드 값 타입 수정 
member.setHomeAddress(new Address("신도시", "신도시구", "12345"));

//2 기본값 타입 컬렉션 수정 
Set<String> favoriteFoods = member.getFavoriteFoods();
favoriteFoods.remove("탕수육");
favoriteFoods.add("치킨");

//3 임베디드 값 타입 컬렉션 수정 
List<Address> addressHistory = member.getAddressHistory(); 
addressHistory.remove(new Address("신도시", "신도시구", "12345"));
addressHistory.add(new Address("신도시", "신도시구", "12345"));
```
#### 1 임베디드 값 타입 수정 
- homeAddress 임베디드 값 타입은 MEMBER 테이블과 매핑했으므로 MEMBER 테이블만 업데이트 한다. 
#### 2 기본값 타입 컬렉션 수정 
- 탕수육을 치킨으로 변경하려면 탕수육을 제거하고 치킨을 추가해야 한다. 자바의 String 타입은 수정할 수 없다. 
#### 3 임베디드 값 타입 컬렉션 수정 
- 값 타입을 불변해야 한다 따라서 컬렉션에서 기존 주소를 수정하고 새로운 주소를 등록했고, 값타입은 quals, hashCode를 꼭 구현해야 한다. 

### 9.5.2 값 타입 컬렉션 제약사항 
- 엔티티는 식별자가 있으므로 엔티티의 값을 변경해도 식별자로 데이터베이스에 저장된 원본데이터를 쉽게 찾아서 변경가능하다. 
- 값 타입은 식별자라는 개념이 없고 단순한 값들의 모음이므로 값을 변경해버리면 데이터베이스에서 저장된 원본 데이터를 찾을 수 없다. 
- 특정 엔티티에 소속된 값 타입은 값이 변경되어도 자식이 소속된 엔티티를 데이터베이스에서 찾고 값을 변경가면 되지만 문제는 값 타입 컬렉션이다. 
- 값타입 컬렉션에 보관된 값 타입들은 별도의 테이블에 보관된다. 따라서 여기에 보관된 값 타입의 값이 변경되면 데이터베이스에 있는 원본 데이터를 찾기 어렵다는 문제가 있다. 
- JPA구현체들은 값 타입 컬렉션에 변경 사항이 발생하면 값타입 컬렉션이 매핑된 테이블의 연관된 데이터를 삭제하고, 현재 값 타입 컬렉션 객체이 있는 모든 값을 데이터 베이스에 저장한다. 
- 실무에서는 값 타입 컬렉션이 매핑된 테이블에 데이터가 많다면 값 타입 컬렉션 대신에 일대다 관계를 매핑해야 한다. 
- 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본키를 구성해야 한다. 따라서 데이터베이스에 기본기 제약 조건으로 인해 컬럼에 null을 입력할 수 없고 값을 중복해서 저장할 수 없는 제약도 있다. 
- 이러한 문제들을 해결하기 위해서는 값 타입 컬렉션 대신에 새로운 엔티티를 만들어서 일대다 관계로 설정하면 된다. 

