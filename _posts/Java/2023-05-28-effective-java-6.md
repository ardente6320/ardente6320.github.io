---
title:  "Effective Java-ITEM 6 불필요한 객체 생성을 피하라"
date:   2023-05-28 10:47:00+0900
categories: [Java]
tags: [Java, Effective Java]
---
<br>

똑같은 기능의 객체를 매번 생성하게 되면 메모리 낭비로 이어질 수 있다.

이런 경우 객체 하나를 재사용하는 편이 나을 때가 많다.

단편적인 예를 들어보자.

```java
String str1 = "Pizza";

String createdStr1 = new String("Pizza");
```

위 String 생성에 대한 예를 생객해본다면, `str1`과 `createdStr1`의 주소값은 다르다.

`new`를 통해 새로운 인스턴스를 지속적으로 생성한다면 이는 쓸데없는 `String`인스턴스가 수백만개 만들어질 수도 있다.

---

여기서 `String`에 대해 잠깐만 설명하자면, 이는 특별한 참조 자료형이다.

`String`은 `new`를 통해 인스턴스를 생성하고 `heap`에서 메모리를 관리한다는 것은 다른 참조 자료형과 다를바 없다.

하지만 다른 참조 자료형과는 다르게 변하지 않는다. 이는 `String Constant Pool`에서 관리하기 때문이다.

해당 `pool`에 기존에 만들어진 `str1`이 저장되어 있어서 만약 `String str2 = "Pizza"`를 하게 된다면 `str2`는 `str1`과 같은 주소를 바라보게 된다.

하지만 `new String`을 하게 된다면 `heap`에 개별 객체가 생성되기 떄문에 `String Constant Pool`에는 들어가지 않는 것이다.

---


`String str1 = "Pizza"` 형식을 사용한다면 같은 JVM 안에서 이와 같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체르 재사용함이 보장된다.

<br>

다음은 정규 표현식을 활용한 예를 나타낸다.

```java
static boolean isDecimalNumber(String s){
    return s.matches("^\\d+$");
}
```

해당 방식의 문제는 `String.matches`를 사용한다는데 있다.

`matches`메서드는 Pattern 인스턴스를 사용하는데, 한 번 쓰고 버려져서 곧바로 GC의 대상이 된다.

이런 성능을 개선하기 위해선 Pattern 인스턴스를 ㅈ어적 초기화를 통해 직접 생성해 캐싱해두고 해당 인스턴스를 재사용하는 방식을 사용하면 성능 개선에 도움이 된다.

```java
public class DecimalNumber{
    private static final Pattern DECIMAL_NUMBER = Pattern.compile("^\\d+$");
    
    public static boolean isDecimalNumber(String s){
        return DECIMAL_NUMBER.matcher(s).matches();
    }
}
```

반복문으로 1000번 반복했을때 재사용한것이 안한것에 비해 월등히 빨라짐을 확인 할 수 있을 것이다.

```java
//측정 방법

long startTimeForNonReuse = System.currentTimeMillis();

for(int i = 0; i < 1000; i++){
    isDecimalNumber(i+"");
}

long estimateTimeForNonReUse = System.currentTimeMillis() - startTimeForNonReuse;

long startTimeForReUse = System.currentTimeMillis();

for(int i = 0; i < 1000; i++){
    DecimalNumber.isDecimalNumber(i+"");
}
long estimateTimeForReUse = System.currentTimeMillis() - startTimeForReUse;

System.out.println(estimateTimeForNonReUse);
System.out.println(estimateTimeForReUse);
```

만약 `DecimalNumber.isDecimalNumber`을 호출하지 않았다면 `Pattern`인스턴스를 초기화한것은 쓸데없이 초기화한 꼴이 될 것이다. 

이를 방지하기위해 메서드가 처음 호출될 때 필드를 초기화하는 지연 초기화로 불필요한 초기화를 없앨 수 있지만,

**오히려 코드를 복잡하게 하고 성능이 크게 개선되지 않기 때문에 권하지는 않는다.**

<br>

한가지 더 예를 들어보겠다.

오토박싱(auto boxing)또한 불필요한 객체를 만들어내는 또다른 예로 들수 있다.

오토박싱이란 기본 타입과 박싱된 기본타입을 섞어 쓸때 자동으로 상호 변환해주는 기술이다.

이때 구분을 완전히 없애주는 건 아니다.

의미상으로는 구분이 안가지만 성능적으로는 차이가 많이 난다.


```java
long sum(){
    Long sum = 0L;

    for(long i = 0; i < Integer.MAX_VALUE; i++){
        sum += i;
    }

    return sum;
} 
```

해당 메소드를 보면 Long 타입으로 선언해서 불필요한 Long 인스턴스가 2^31 개 만큼 생성 될것이다.

이때 Long 을 primitive type인 long으로 바꿔주면 성능이 월등히 향상될 것이다.

<br>

---

객체 생성은 값이 비싸니 생성을 피하자라는 내용은 이번 주제와 무관하다.

JVM에서 객체 생성하고 회수하는 일은 크게 부담되지 않는다.

오히려 명확성, 간결성 기능을 위한 객체 생성은 좋은 행위이다.

단 기존 객체를 재사용해야한다면 새로운 객체를 만들지 말라는 의미이다.

방어적 복사를 하는 경우를 생각하면 재사용 했을 때의 피해가 새롭게 생성했을 때에 비해 크다.

하지만 불필요한 객체 생성은 그저 코드 형태와 성능에 영향을 준다는 것을 명심하자.