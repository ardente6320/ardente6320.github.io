---
title:  "Effective Java-ITEM 10 equals는 일반 규악을 지켜 재정의하라"
date:   2023-07-17 22:17:11+0900
categories: [Java]
tags: [Java, Effective Java, equals]
---
<br>

`equals` 메소드는 재정의하기 쉬워 보이지만 정말 조심해서 재정의 해야한다.

---

#### **<span style="color:#ef5369">재정의 하지 말아야 하는 경우</span>**

<br>

**1. 각 인스턴스가 본질적으로 고유한 경우**

값을 표현하는게 아닌 동작하는 개체를 표현한 인스턴스는 동일한 인스턴스가 애초에 없다.

그래서 `Object`의 `equals`만으로 충분하다. ex) Thread

**2. 인스턴스의 논리적 동치성(Logical Equality)을 검사할 일이 없는 경우**

값이 동등한지 비교할 일이 없다면 논리적 동치성(Logical Equality)를 검사할 일이 없고, `Object`의 `equals`로 충분하다.

**3. 상위 클래스에서 재정의한 `equals`가 하위 클래스에도 딱 들어맞을 경우**

상위 클래스에서 구현한 `equals`로 충분한 경우 하위 클래스에서 재정의 할 필요없이 상위 클래스의 `equals`를 그대로 상속받아 사용한다.

ex)
- `Set`구현체의 경우 `AbstractSet`의 `equals`
- `List`구현체의 경우 `AbstractList`의 `equals`
- `Map`구현체의 경우 `AbstractMap`의 `equals`

**4. 클래스가 `private`이거나 `default(package-private)`이고 `equals`를 호출할 일이 없는 경우**

```java
@Override 
public boolean equals(Object o) {
	throw new AssertionError(); // 호출 금지!
}
```

위와 같이 구현하여 호출하는 것을 막도록 한다.

---

#### **<span style="color:#ef5369">재정의를 해야하는 경우</span>**

<br>

객체 식별정(물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데,

상위 클래스의 `equals`가 논리적 동치성을 비교하도록 재정의되지 않았을 경우이다.

예를 들어 `Integer`와 `String`처럼 값 클래스를 비교할 경우이다.

객체가 같은 경우가 아닌 두 값이 같은지 비교하게 된다.

---

#### **<span style="color:#ef5369">재정의 일반 규약</span>**

<br>

**1. 반사성(reflexivity)**

`null`이 아닌 모든 참조 값 `x`에 대해, `x.equals(x)`는 `true`다.

**2. 대칭성(symmetry)**

`null`이 아닌 모든 참조값 `x`,`y`에 대해, `x.equals(y)`가 `true`면 `y.equals(x)`도 `true`다.


**3. 추이성(transitivity)**

`null`이 아닌 모든 참조 값 `x`,`y`,`z`에 대해, `x.equals(y)`와 `y.equals(z)`가 `true`이면 `x.equals(z)`도 `true`다.

**4. 일관성(consistency)**

`null`이 아닌 모든 참조값 `x`,`y`에 대해, `x.equals(y)`를 반복해서 호출하면 항상 `true`를 반환하거나 항상 `false`를 반환한다.

**5. `null` 아님**

`null`이 아닌 모든 참조 값 `x`에 대해, `x.equals(null)`은 항상 `false`다.

의도하지 않았지만 실수로 `NullPointerException`을 던지지 않도록 방어해야한다.

```java
//명시적 검사
@Override
public boolean equals(Object o){
    if(o == null)
        return false;
}
```
```java
//묵시적 검사
@Override
public boolean equals(Object o){
    if(!(o instanceof MyClass))
        return false;
    
    MyClass myClass = (MyClass) o;
    ...
}
```

위 코드와 같이 명시적으로 `null`을 검사하는 것 보단 

`instanceof`연산자를 활용해 올바른 타입인지 검사하는 것이 났다.

`equals`가 타입을 확인하지 않으면 잘못된 타입이 들어오는 경우 `ClassCastException`을 던져 규약 위배가 된다.

그리고 `instanceof`연산자를 통해 비교할 경우 `null`또한 처리되기 때문에 묵시적 검사가 더욱 났다.

---

#### **<span style="color:#ef5369">양질의 `equals`메소드 구현 방법</span>**

<br>

**1. `==`연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.**

자기 자신이면 `true`를 반환한다.

```java
if(o === this)
    return true;
```

**2. `instanceof`연산자로 입력이 올바른 타입인지 확인한다.**

올바르지 않는 타입인 경우(또는 `null`) `false`를 반환한다.

```java
if(!(o instanceof Person))
    return false;
```

**3. 입력을 올바른 타입으로 형 변환한다.**

```java
Person person = (Person) o;
```

**4. 입력 객체와 자기자신의 대응되는 핵심필드들이 모두 일치하는지 하나씩 검사한다.**

하나라도 다른 경우 `false`를 반환한다.

비교할 때 어떤 필드를 비교하느냐에 따라 성능을 좌우하기도 한다.

성능을 고려한다면 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교한다.

```java
return person.age == age && person.name.equals(name);
```

**5. 대칭성, 추이성, 일관성을 검증한다.**

**6. `equals`를 재정의할 땐 `hashCode`도 반드시 재정의한다.**

**7. 너무 복잡하게 해결하려 하지 말자.**

필드의 동치성만 검사해도 규약을 어렵지 않게 지킬 수 있다.

**8. `Object`외 타입을 매개변수로 받는 `equals`메소드는 선언하지 말자.**

타입을 구체적으로 명시한 `equals`는 오히려 해가된다.

```java
//잘못된 예 - 컴파일 되지 않는다.
@Override
public boolean equals(MyClass a){
        ...
}
```

위 코드는 `Object.equals`를 재정의한 것이 아닌 다중정의를 한 것이다.

이 메소드는 `@Override`어노테이션이 긍정오류를 내개 하고 보안 측면에서도 잘못된 정보를 준다.

---

위 코드를 하나의 메소드로 나타내면 다음과 같다.

```java
class Person{
    private String name;
    private int age;
    
    public Person(String name, int age){
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object o){
        if(o == this)
            return true;
        if(!(o instanceof Person))
            return false;
        
        Person person = (Person) o;
        
        return person.age = age && person.name.equals(name);
    }
}
```

---

결론적으로 꼭 필요한 경우가 아니라면 `equals`는 재정의하지 말자.

만약 재정의하게 된다면 클래스의 핵심 필드를 모두 빠짐없이 확인해야하고, 다섯가지 규약을 확실히 지켜가며 비교해야한다.
