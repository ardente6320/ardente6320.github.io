---
title:  "Effective Java-ITEM 4 인스턴스화를 막으려거든 private 생성자를 사용하라"
date:   2022-12-02 23:01:01
categories: [Java]
tags: [Java, Effective Java]
---
<br>

정적 메소드나 정적 필드를 사용하여 유틸리티 클래스를 작성하는 경우가 많다.

예를 들어 `java.lang.Math`와 `java.uti.Arrays` 같은 클래스는 기본 타입과 배열 관련 메소드들을 모아 놓았다.

또한 `java.util.Collections` 처럼 특정 인터페이스의 객체를 생성해주는 정적 팩토리 메소드를 모아 놓은 것도 있다.

이런 클래스의 경우 인스턴스화를 하여 사용하려고 설계한 것이 아니다.

생성자를 명시하지 않으면 컴파일러가 파라미터가 없는 `public` 생성자를 default로 만들어준다.

이를 방지하여 인스턴스화가 되지 않도록 해야한다.

<br>

#### **<span style="color:#ef5369">인스턴스화 막는 방법</span>**

```java
public class UtilityClass{
    //기본생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
    private UtilityClass(){
        throw new AssertionError();
    }

    ... //나머지 코드 생략
}
```

추상클래스를 만드는 것으로는 인스턴스화를 막을 수 없다. 하위 클래스가 상속받아 인스턴스화할 수 있기 때문에 적절한 방법이 아니다.

인스턴스화를 막으려면 `private` 생성자를 생성해주면 막을 수 있다.

클래스를 상속해서 인스턴스화를 하려고 해도 부모 클래스의 생성자가 호출되어야 하기 때문에 불가능하다.

꼭 `AssertionError()`를 던질 필요는 없지만, 클래스 내부에서 실수로라도 생성자가 호출되어 인스턴스화가 되는 것을 막아준다.