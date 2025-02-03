# 9장 값타입 
## 9.3 값 타입과 불변 객체 
- 값 타입은 복잡한 객체 사상을 조금이라도 단순화하려고 만든 개념이다 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다. 

### 9.3.1 값타입 공유 참조 
- 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험하다. 
```java 
member1.setHomeAddress(new Address("OldCity"));
Address address = member1.getHomeAddress(); 

address.setCity("NewCity");
member2.setHomeAddress(address); 
```
- 회원2에 새로운 주소를 할당할려고 회원 1의 주소를 그래도 참조해서 사용했다. 
- 이 코드를 실행하면 회원 2의 주소만 NewCity로 변경되길 기대했지만 회원1의 주소도 NewCity로 변경되어 버린다. 
- 회원1과 회원2가 같은 address 인스턴스를 참조하기 때문이다. 이러한 공유 참조로 인해 발생하는 버그는 참아내기 어렵다.
- 이렇게 무언가를 수정했는데 전혀 예상치 못한 곳에서 문제가 발생하는 것을 side effect이라 한다. 이러한 부작용을 막기 위해서는 값을 복사해서 사용하면 된다. 

### 9.3.2 값 타입 복사 
- 값 타입의 실제 인스턴스인 값을 공유해서 사용하는 것은 위험하기 때문에 값(인스턴스)을 복사해서 사용해야 한다. 

```java
member1.setHomeAddress(new Address("OldCity")); 
Address address = memeber1.getHomeAddress(); 

Address newAddress = address.clone(); 

newAddress.setCity("newCity");
member2.setNewAddress(newAddress);

```
- 항상 값을 복사해서 사용하면 공유참조로 인해 발생하는 부작용을 피할 수 있다. 
- 문제는 임베디드 타입 처럼 직접 정의한 값타입은 자바 기본 타입이 아니라 객체 타입 이라는 것이다. 
- 객체를 대입할 때 마다 인스턴스를 복사해서 대입하면 공유참조를 피할 수 있다. 문제는 **복사하지 않고 원본의 참조 값을 직접 넘기는 것을 막을 방법이 없다는 것이다.** 
- 객체의 공유 참조를 피할 수 없다. 따라서 근본적인 해결책이 필요한데 가장 단순한 방법은 객체의 값을 수정하지 못하게 만들면 된다. 
- Address 객체의 setCity()같은 수정 메소드를 모두 제거하자. 

### 9.3.3 불변객체
- 값 타입은 부작용 걱정 없이 사용할 수 있어야 한다. 부작용이 일어나면 값 타입이라 할 수없다. 
- 객체를 불변하게 만들면 값을 수정할 수 없으므로 부작용을 원천 차단 할 수 있다. 따라서 값 타입을 될수 있으면 **불변 객체**로 설계해야 한다. 
- 한번 만들면 절대 변경할 수 없는 객체를 불변 객체라고 한다. 불변 객체의 값은 조회는 가능하지만 수정을 할 수 없다. 
- 불변 객체도 결국 객체이기 때문에 인스턴스의 참조값 공유를 피할 수 없다. 하지만 참조값을 공유해도 인스턴스의 값을 수정할 수 없으므로 부작용이 발생하지 않는다. 
- 불변 객체를 구현하는 다양한 방법이 있지만 가장 간단한 방법은 생성자로만 값을 설정하고 수정자를 만들지 않으면 된다. 

```java
@Embeddable 
public class Address {
    
    private String city 

    protected Address() //기본 생성자 

    public Address(String city){
        this.city = city;
    }

    public String getCity(){
        return city; 
    }
}
```
