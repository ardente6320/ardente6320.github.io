---
title:  "Effective Java-ITEM 11 equals를 재정의하려거든 hashCode도 재정의하라"
date:   2023-07-25 12:51:30+0900
categories: [Java]
tags: [Java, Effective Java, equals, hashCode]
---
<br>

`equals`를 재정의한 클래스 모두에서 `hashCode`도 재정의해야 한다.

--- 

#### **<span style="color:#ef5369">`Object` 명세에 대한 규약</span>**

1. `equals`비교에 사용되는 정보가 변경되지 않았다면, 서비스가 실행되는 동안 해당 객체의 `hashCode`메소드는 일관되게 항상 같은 값을 반환해야한다.
2. `equals(Object)`가 두 객체를 같다고 판단했다면, 두 객체의 `hashCode`는 똑같은 값을 반환해야한다.
3. `equals(Object)`가 두 객체를 다르다고 판단했더라도, 두 객체의 `hashCode`가 서로 다른 값을 반환할 필요는 없다. 단 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

---

`hashCode`재정의를 할 때는 논리적으로 같은 객체는 같은 해시코드를 반환해야한다.

`euqals`는 물리적으로 다른 두 객체를 논리적으로는 같다고 할 수는 있다.

하지만 `Object`의 기본 `hashCode`메소드는 이둘이 전혀 다르다고 판단하여, 서로 다른 값을 반환한다.

```java
Map<Person, String> personMap = new HashMap<>();

//Person(age,gender,serialNumber)
personMap.put(new Persion(30,"M","123456"),"게롤트");
```

해당 코드 다음에 `personMap.get(new Persion(30,"M","123456"))`를 하면 "게롤트"가 나올 것 같지만, 실제로는 `null`을 반환한다.

여기서 사용한 인스턴스는 총 2개이다. 

한개는 `personMap`에 `put`할 때와 `get`할 때이다.

하지만 `Person` 클래스는 `hashCode`를 재정의하지 않았기 때문에 논리적으론 같지만 서로 다른 `hashCode`를 반환하여 다른 객체로 인식하게 된다.

---

#### **<span style="color:#ef5369">좋은 `hashCode` 재정의 방법</span>**

1. `int` 변수인 `result`를 선언한 후 값을 `c`로 초기화한다. 
   - 이 때, `c`는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드이다.
   - 여기서 핵심 필드는 `equals` 비교에 사용되는 필드를 말한다.
2. 해당 객체의 나머지 핵심 필드인 `f` 각각에 대해 다음 작업을 수행한다.
   1. 해당 필드의 해시코드 `c` 를 계산한다.
      - 기본 타입 필드라면, `Type.hashCode(f)`를 수행한다. 여기서 `Type`은 해당 기본타입의 박싱 클래스다.
      - 참조 타입 필드면서, 이 클래스의 `equals`메소드가 이 필드의 `equals`를 재귀적으로 호출하여 비교한다면, 이 필드의 `hashCode`를 재귀적으로 호출한다.
      - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용한다.
   2. 단계 2.1에서 계산한 해시코드 `c`로 `result`를 갱신한다. 
      - `result = 31 * result + c`; 
3. `result`를 반환한다.

파생 필드는 해시코드 계산에서 제외해도 된다.

즉, 다른 필드로 부터 계산해낼 수 있는 필드는 모두 무시해도 된다.

또한 `equals`비교에 사용되지 않은 필드는 반드시 제외해야 한다.

곱할 숫자를 31로 정한 이유는 31이 홀수이면서 소수(prime number)이기 때문이다.

만약 이 숫자가 짝수이고 overflow가 발생한다면 정보를 잃게 된다.

```java
@Override
public int hashCode(){
    int result = Integer.hashCode(age);
    result = 31 * result + gender.hashCode();
    result = 31 * result + serialNumber.hashCode();
    return result;
}
```

위 코드를 보면 `Person`인스턴스의 핵심 필드 3개만을 사용해 간단하게 계산해 낸다.

이 과정 속에 비결정적 요소는 없으므로 같은 해시코드를 가질 것이 확실하다.

---
`Objects`클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메소드인 `hash`를 제공한다.

간단하지만 속도는 위 코드보다 느리다. 따라서 `hash`메소드는 성능에 민감하지 않은 경우만 사용하도록 하자.

```java
@Override
public int hashCode(){
    return Objects.hash(age, gender, serialNumber);
}
```

---

#### **<span style="color:#ef5369">`hashCode`의 캐싱과 지연 초기화</span>**

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 캐싱하는 방식을 고려해야한다.

- 이 타입의 객체가 주로 해시의 키로 사용될 것 같아면 인스턴스가 만들어질 때 해시코드를 계산해둬야 한다.

해시의 키로 사용되지 않는 경우라면 `hashCode`가 처음 불릴 때 계산하는 지연 초기화를 하면 좋다.
- 필드를 지연 초기화하려면 thread-safe하게 동기화에 신경 써야한다.

**해시코드를 지연초기화하는 `hashCode`메소드 - thread-safe까지 고려해야 한다.**
```java
class Person{
    private int age;
    private String gender;
    private String serialNumber;
    
    private int hashCode; // 자동으로 0으로 초기화된다.

    public Person(int age, String gender, String serialNumber){
        this.age = age;
        this.gender = gender;
        this.serialNumber = serialNumber;
    }

    @Override
    public int hashCode() {
        int result = hashCode;
        
        if(result == 0){
            result = Integer.hashCode(age);
            result = 31 * result + gender.hashCode();
            result = 31 * result + serialNumber.hashCode();
        }
        
        return result;
    }
}
```

---

`equals`를 재정의 할때는 반드시 `hashCode`도 재정의하자.

그렇지 않는 경우 예기치 못한 오류를 만날 수있다. 

또한 `Object`의 API 문서에 기술된 일반 규약을 따라야 하고, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야한다.

자동으로 `equals`와 `hashCode`를 생성해주는 `AutoValue Framework`도 있으니 참고하도록 하자.
