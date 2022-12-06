---
title:  "Effective Java-ITEM 3 싱글턴임을 보증하라"
date:   2022-12-01 10:34:25
categories: [Java]
tags: [Java, Effective Java, Singleton]
---
<br>

#### **<span style="color:#ef5369">싱글턴이란</span>**

싱글턴(Singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

예시로 함수와 같은 stateless 객체나 설계상 유일해야하는 시스템 컴포넌트를 들 수 있다.

---

#### **<span style="color:#ef5369">싱글턴 생성 방법</span>**

<br>

##### **<span style="color:#ef5369">1. `public static final` 필드 방식</span>**

```java
public class CustomProperty {
    public static final CustomProperty INSTANCE = new CustomProperty();

    private CustomProperty{
        ...
    }

    public void readPropertyFromFIle(){
        ...
    }
}
```

`private` 생성자는 `public static final` 필드인 `CustomProperty.INSTANCE`를 초기화할 때 딱 한번만 호출된다.

`public`이나 `protected` 생성자가 없으므로 해당 인스턴스가 전체 시스템에서 하나뿐임을 보장한다.

클라이언트는 해당 인스턴스를 변경할 수 없지만, 권한이 있는 클라이언트는 리플렉션인 `AccessibleObject.setAccessible()`를 사용하여 `private` 생성자를 호출할 수 있다.

이런 공격을 방어하려면 2번째 객체가 생성되려 할때 예외를 던지게 하면 된다.

```java
public class CustomProperty {
    public static final CustomProperty INSTANCE = new CustomProperty();

    private CustomProperty{
        if(INSTANCE != null){
            throw new IllegalStateException("Only one instance may be created");
        }
        ...
    }
}
```

##### **<span style="color:#ef5369">`public static final` 필드 방식의 장점</span>**


  - **<span style="color:#ef5369">해당 클래스가 싱글텀임이 API에 명백히 들러난다.</span>**
  - **<span style="color:#ef5369">간결함</span>**

<br>

##### **<span style="color:#ef5369">2. 정적 팩터리 메서드 방식</span>**

```java
public class CustomProperty {
    private static final CustomProperty INSTANCE = new CustomProperty();

    private CustomProperty{
        if(INSTANCE != null){
            throw new IllegalStateException("Only one instance may be created");
        }
        ...
    }

    public static CustomProperty getInstance(){
        return INSTANCE;
    }

    public void readPropertyFromFIle(){
        ...
    }
}
```

`CustomProperty.getInstance()`는 항상 같은 객체의 참조를 반환하기 때문에 제2의 인스턴스는 결코 만들어지지 않는다. (리플렉션 예외는 동일하게 적용된다.)

##### **<span style="color:#ef5369">정적 팩더리 메서드 방식의 장점</span>**

  - **<span style="color:#ef5369">API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.</span>**
  - **<span style="color:#ef5369">정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.</span>**
  - **<span style="color:#ef5369">정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다.</span>**

이러한 장점들이 굳이 필요하지 않다면 `public static final` 필드 방식이 좋다.

---

#### **<span style="color:#ef5369">싱글턴 직렬화(Serialization)</span>**

싱글턴 클래스를 직렬화할때 주의할 점이 있다.

직렬화 구현 시 단순히 `serializable`를 구현하는 것만으로는 싱글턴이 완벽히 보장되진 않는다.

직렬화된 싱글턴 객체를 다시 역직렬화하면 새로운 인스턴스가 생성되기 때문이다.

이를 예방하기 위해서는 `readResolve` 메서드를 추가해야한다.

```java
public class CustomProperty implements Serializable{
    private static final CustomProperty INSTANCE = new CustomProperty();

    private CustomProperty{
        if(INSTANCE != null){
            throw new IllegalStateException("Only one instance may be created");
        }
        ...
    }

    public static CustomProperty getInstance(){
        return INSTANCE;
    }

    private Object readResolve() {
        // 기존 instance를 반환하고, 가짜 instance는 GC에 의해 반환된다.
        return INSTANCE;
    }

    public void readPropertyFromFIle(){
        ...
    }
}   
```

---

#### **<span style="color:#ef5369">Enum 방식의 싱글턴</span>**

`Enum` 방식의 싱글턴은 `public static final` 필드 방식과 비슷하다.

하지만 더 간결하고, 추가 노력 없이 직렬화 할 수 있다. 

그리고 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.

단, 만들려는 싱글턴이 `Enum` 외의 클래스를 상속해야 한다면 사용할 수 없다.

```java
enum CustomProperty {
    Number("number");

    private final String identifier;

    public final String getIdentifier() {
        return identifier;
    }

    private CustomProperty(String identifier) {
        this.identifier = identifier;
    }
}
```