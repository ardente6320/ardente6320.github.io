---
title:  "Effective Java-ITEM 5 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라"
date:   2022-12-05 15:11:30
categories: [Java]
tags: [Java, Effective Java]
---
<br>

많은 클래스가 하나 이상의 자원에 의존한다.

책에서는 맞춤법 검사기를 예시로 들고 있다. 

이렇게 의존성을 가지고 있는 클래스를 정적 유틸리키 클래스 또는 싱글턴으로 구현하는 경우가 있다.

#### **<span style="color:#ef5369">정적 유틸리티 클래스 방식</span>**

```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {} //객체 생성 방지

    public static boolean isValid(String word) { ... }

    public static List<String> suggestions(String typo) { ... }
}
```

<br>

#### **<span style="color:#ef5369">싱글턴 방식</span>**

```java
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker() {}

    public static final SpellChecker INSTANCE = new SpellChecker();

    public boolean isValid(String word) { ... }

    public List<String> suggestions(String typo) { ... }
}
```

위 두가지 방식은 사전 클래스인 `Lexicon`을 다른 사전으로 교체하려면 코드 자체를 수정해야한다.

실전에서는 사전이 언어별로 따로 있고, 특수 어휘용, 테스트 용 등 다양하게 있을 수 있다.

**사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.**

대신 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 **의존 객제 주입 패턴**을 사용하여 해결할 수 있다.

이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.

<br>

#### **<span style="color:#ef5369">의존 객체 주입 패턴</span>**

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }

    public List<String> suggestions(String typo) { ... }
}
```

위 코드에서는 하나의 자원을 사용하지만 여러개의 자원을 사용해도 상관없다. 

또한 불변(final)을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다.

**의존 객체 주입 패턴**은 정적 팩터리 메서드 패턴(ITEM 1), 빌더 패턴(ITEM 2) 모두 똑같이 응용할 수 있다.

<br>

#### **<span style="color:#ef5369">팩터리 매서드 패턴</span>**

**의존 객체 주입 패턴**의 변형으로 생성자에 자원 팩터리를 넘겨주는 방식이다.

  - 팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다.  

java 8에서부터 사용가능한 `Supplier<T>` 인터페이스를 활용한 예이다.

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Supplier<Lexicon> dictionarySupplier) {
        this.dictionary = dictionarySupplier.get();
    }

    public boolean isValid(String word) { ... }

    public List<String> suggestions(String typo) { ... }
}

//Main class 사용
public class Main{
    public static void main(String[] args){
        SpellChecker englishSpellChecker = new SpellChecker(new Supplier<Lexicon>() {
            @Override
            public Lexicon get() {
                return new EnglishDictionary();
            }
        });

        SpellChecker koreanSpellChecker = new SpllChecker(() -> new KoreanDictionary());
    }
}
```

`Supplier<T>`는 [함수형 인터페이스](#functional-interface)이므로 람다 표현식 표현이 가능하다.
  - _[<U>Supplier API 명세서</U>](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html)_

---

#### **<span style="color:#ef5369">참고</span>**

# <span style="display:none;">Functional Interface</span>
1. 함수형 인터페이스란?
   - 추상 메서드가 오직 하나인 인터페이스를 의미한다.
   - default 메서드와 static 메서드는 여러개 존재해도 관계 없다.
   - @FunctionalInterface 어노테이션을 사용한다.(없어도 가능하지만 인터페이스 검증과 유지보수를 위해 붙임)
   - 예시 코드
     ```java
        @FunctionalInterface
        interface CustomFunctionalInterface<T> {
            //오직 한개의 추상 메서드
            T get();

            default void printDefaultMessage(){
                System.out.println("this is Defualt Method");
            }
            
            static void printStaticMessage(){
                System.out.println("this is Static Method");
            }
        }
     ```
---