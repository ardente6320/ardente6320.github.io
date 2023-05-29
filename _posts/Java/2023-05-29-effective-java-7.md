---
title:  "Effective Java-ITEM 7 다 쓴 객체 참조를 해제하라"
date:   2023-05-29 20:30:15+0900
categories: [Java]
tags: [Java, Effective Java]
---
<br>

C, C++의 경우 메모리를 직접 관리해야한다. 

하지만 Java는 GC를 갖춘 언어이다. 즉 GC가 다 쓴 객체를 알아서 회수해간다.

그러나 GC가 알아서 회수해간다고 해서 메모리 관리를 신경 쓸 필요가 없는 것은 아니다.

```java
public class Stack{
    private Object[] frame;
    private int size;

    public Stack(){
        frame = new Object[10];
    }

    public void push(Object obj){
        if(frame.length == index){
            frame = Arrays.copyOf(frame, 2 * frame.length + 1);
        }

        frame[size++] = obj;
    }

    public Object pop(){
        if(size < 0){
            throw new IndexOutOfBoundsException();
        }
        return frame[--size];
    }
}
```

위 코드를 보면 간단한 스택에 대해 작성한 코드이다.

보기에는 큰 문제가 없어보이긴 하나, 오래 실행하면 할수록 **메모리 누수**로 이어질 수 있다.

심하면 디스크 페이징이나 `OutOfMemoryError`를 발생해 예기치 못한 종료가 나올 수 있다.

<br>

왜 이런 **메모리 누수**가 나오는 것일까?

`pop()` 메소드를 보면 다 쓴 객체에 대해 참조를 해제하지 않고 메모리에 들고 있다.

새로운 것이 들어오지 않는 이상 해제 되어도 무방한 데이터를 계속 들고 있는 것이다.

이런 상황은 다음과 같이 `pop()`메소드를 작성하면 해결된다.

```java
public Object pop(){
    if(size < 0){
        throw new IndexOutOfBoundsException();
    }
    Object obj = frame[--size];
    frame[size] = null;

    return obj;
}
```

다 쓴 참조를 `null`처리 하게 되면 GC의 처리 대상이되어 처리될 것이고

또 만약에 참조하게 되더라도 `NullPointerException`이 발생하여 종료되어 잘못된 참조를 막을 수 있다.


이런 스택과 같은 클래스는 자기 메모리를 직접 관리하기 때문에 **메모리 누수**에 취약할 수 있다.

이렇게 자기 메모리를 직접 관리하여 **메모리 누수**에 주의 해야한다.


---
**메모리 누수**는 겉으로 잘 드러나지 않아 오랫동안 잠복할 수도 있다. 

그래서 꼭 예방하는 방법을 익혀서 누수를 관리할 수 있도록 하자.