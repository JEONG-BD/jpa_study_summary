# 9장 값타입 
- JPA의 데이터 타입을 크게 분류하면 엔티티 타입과 값타입으로 나눌 수 있다. 
- 엔티티 타입은 @Entity로 정의한느 객체이고 값 타입은 int, Integet, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체를 말한다. 
- 엔티티 타입은 식별자를 통해 지속해서 추적할 수 있지만 값 타입은 식별자가 없고 숫자가 문자같은 속성만 있으므로 추적할 수 없다. 
- 값 타입은 기본값타입, 임베디드 타입, 컬렉션 값 타입으로 나눌 수 있다. 

## 9.1 기본 값 타입 
```java
@Entity 
public class Member{
    @Id
    @GeneratedValue 
    private Long id; 
    private String name; 
    private int age; 
}

```
- String, int 가 값 타입이다. 
- Member 엔티티는 id라는 식별자 값도 가지고 있고 생명주기도 있지만 값 타입인 name, age속성은 식별자 값도 없고 생명주기도 회원 엔티티에 의존한다. 

## 9.2 임베디드 타입 (복합 값 타입)
- 새로운 값 타입을 직정 정의해서 사용할 수 있는데 이것을 JPA에서는 이것을 임베디드 타입 이라고 한다. 
- 중요한 것은 직접 정의한 임베디드 타입도 int, String 처럼 기본 값 타입이라는 것이다. 

```java
@Entity 
public class Member{

    @Id
    @GeneratedValue 
    private Long id;

    private String name; 

    // 근무기간 
    @Temporal(TemporalType.DATE) java.util.Date startDate; 
    @Temporal(TemporalType.DATE) java.util.Date endDate; 
    
    // 집주소
    private String city;
    private String street; 
    private String zipCode; 

}

@Entity 
public class Member{

    @Id
    @GeneratedValue 
    private Long id;

    private String name; 
    
    @Embedded Period workPeriod;
    @Embedded Address homeAddress;
}

@Embeddeable
public class Period {
        // 근무기간 
    @Temporal(TemporalType.DATE) java.util.Date startDate; 
    @Temporal(TemporalType.DATE) java.util.Date endDate; 
    
    public boolean isWork(Date date){

    }
}

@Embeddable 
public class Address {

    @Column(name= "city")
    private String city;
    private String street; 
    private String zipCode; 
}
```
- 새로 정의한 값 타이블은 재사용할 수 있고 응집도도 아주 높다. 
- 임베디드 타입을 사용하려면 두가지 어노테이션이 필요하다. 
- @Embeddable : 값 타입을 정의하는 곳에 표시 
- @Embedded : 값 타입을 사용하는 곳에 표시 
- 임베다드 타입은 기본 생성자가 필수이다. 


### 9.2.1 임베디드 타입과 테이블 매핑 
- 임베디드 타입은 엔티티의 값일 뿐이다. 따라서 값이 속한 엔티티의 테이블에 매핑한다. 
- 임베디드 타입 덕분에 객체와 테이블을 아주 세밀하게 매핑하는 것이 가능하다. 

### 9.2.2 임베디드 타입과 연관관계 
- 입베디드 타입은 값 타입을 포함하거나 엔티티를 참조할 수 있다. 

```java
@Entity 
public class Member {
    @Embedded Address address ;
    @Embedded PhoneNumber phoneNumber; 
}

@Embeddable 
public class Address{
    String street; 
    String city; 
    String state; 
    @Embedded ZipCode zipcode;

}

@Embeddable 
public class Zipcode {
    String zip;
    String plusFour;
}

@Embeddable 
public class PhoneNumber{
    String areaCode; 
    String localNumber; 
    @ManyToOne PhoneServiceProvider provider; 

}

@Entity 
public class PhoneServiceProvider provider{
    @Id String name;  
}

```

### 9.2.3 @AttributeOverride: 속성 재정의 
- 임베디드 타입에 정의한 매핑정보를 재정의하려면 엔티티에 @AttributeOverride를 사용하면 된다. 
```java
@Entity
public class Member{
    @Id 
    @GeneratedValue
    private Long id; 
    private String name; 

    @Embedded Address homeAddress; 
    @Embedded Address companyAddress; 

}

@Entity
public class Member{
    @Id 
    @GeneratedValue
    private Long id; 
    private String name; 

    @Embedded Address homeAddress; 
    @Embedded 
    @AttributeOverrides({
        @AttributeOverride(name = "city", column=@Column(name = "COMPANY_CITY")), 
        @AttributeOverride(name = "street", column=@Column(name = "COMPANY_STREET")), 
        @AttributeOverride(name = "zipcode", column=@Column(name = "COMPANY_ZIPCODE")), 
        
    })
    Address companyAddress; 
}

```
- @AttributeOverride를 사용하면 어노테이션을 너무 많이 사용해서 엔티티 코드가 지저분해지지만, 한 엔티티에 같은 임베디드 타입을 중복해서 사용하는 일이 많지 않다. 

### 9.2.4 임베디드 타입과 null  
- 임베디트 타입이 null이면 매핑한 컬럼값은 모두 null이 된다. 
```java
member.setAddress(null)//null 입력 
em.persist(member);
// 회원 테이블의 주소와 관련된 CITY, STREET, ZIPCODE컬럼 값은 모두 null이 된다. 
```
