---
title:  "Effective Java-ITEM 10 equals는 일반 규악을 지켜 재정의하라"
date:   2023-07-17 22:17:11+0900
categories: [Java]
tags: [Java, Effective Java, equals]
---
<br>

`equals` 메소드는 재정의하기 쉬워 보이지만 정말 조심해서 재정의 해야한다.

#### **<span style="color:#ef5369">재정의 하지 말아야 하는 경우</span>**

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

#### **<span style="color:#ef5369">재정의를 해야하는 경우</span>**

객체 식별정(물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데,

상위 클래스의 `equals`가 논리적 동치성을 비교하도록 재정의되지 않았을 경우이다.

예를 들어 `Integer`와 `String`처럼 값 클래스를 비교할 경우이다.

객체가 같은 경우가 아닌 두 값이 같은지 비교하게 된다.

#### **<span style="color:#ef5369">재정의 일반 규약</span>**

1. 반사성(reflexivity)