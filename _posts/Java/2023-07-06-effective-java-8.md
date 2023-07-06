---
title:  "Effective Java-ITEM 8 finalizer와 cleaner 사용을 피하라"
date:   2023-07-06 12:33:23+0900
categories: [Java]
tags: [Java, Effective Java]
---
<br>

자바가 제공하는 두지 객체 소멸자가 있다. 

`finalizer`와 `cleaner`이다. 하지만 모두 사용을 하지 말아야 한다.

소멸자를 사용하는 대신 자바에서 접근할 수 없게 된 객체를 회수하는 **Garbage Collector**에 
전적으로 맡겨야 한다.

그리고 비메모리 자원 회수를 위해서 `try-with-resources`와 `try-finally`를 사용해야한다.

---
### **<span style="color:#ef5369">사용하지 말아야 하는 이유</span>**

> 제때 실행되어야 하는 작업은 절대 할수 없다.

`finalizer`와 `cleaner`는 즉시 수행 된다는 보장이 없다.

시스템은 동시에 열 수 있는 파일 개수가 한계가 있다.

만약 `finalizer`나 `cleaner`실행을 게을리해서 파일을 계속 열어둔 다면,

결국 새로운 파일을 열지 못해 프로그램이 실패할 수 있다.


>실행 시간을 예측 할 수 없다.

`finalizer`와 `cleaner`를 얼마나 신속히 수행할지는 **Garbage Collector**의 알고리즘과 구현 방식에 따라 다르다.

만약 테스트 환경과 운영 환경의 JVM이 달라 `finalizer`나 `cleaner` 처리가 굼뜨게 되면

알수 없는 `OutOfMemoryError`를 만나고 프로그램이 실패할 수 있다.


>수행 여부를 보장하지 않는다.

종료 작업을 전혀 수행하지 못한 채 프로그램이 종료될 수 있다. 

따라서 생애주기와 상관없는 상태를 영구적으로 수정하는 작업에서는 절대 사용해선 안된다.

`System.gc`나 `System.runFinalization`은 가능성을 높여줄 뿐 보장하진 않기 때문에 

해당 메소드로 강제 호출 시도를 하지 말아야 한다.


> 심각한 성능 문제를 동반한다.

`AutoCloseable` 객체를 생성하고 `try-with-resources`로 처리한 것과 `finalizer`로 처리하는 것의 속도 차이는 5배 이상으로 나타난다.

미쳐 회수하지 못한 객체를 회수하기 위해 안전망으로 두는 것은 좋지만,

역시 그의 대가로 성능이 약 5배 가량 느려진다는 것을 말한다.


> finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다.

생성자나 직렬화 과정에서 예외가 발생하면 생성되다 만 객체에서 악의적인 하위 클래스의 `finalizer`가 수행될 수 있다.

또한 `finalizer`는 정적 필드에 자신의 참조를 할당하여 **Garbage Collector**가 수집하지 못하게 막을 수 있다.

방어를 하기 위해서 아무 일도 하지 않는 finalize 메소드를 만들고 final로 선언하면 된다.

