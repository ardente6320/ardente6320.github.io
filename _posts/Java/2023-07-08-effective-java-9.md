---
title:  "Effective Java-ITEM 9 try-finally 보다는 try-with-resources를 사용하라"
date:   2023-07-08 16:35:02+0900
categories: [Java]
tags: [Java, Effective Java]
---
<br>

자바에서는 `close()`를 호출해 직접 닫아야하는 자원이 많다.

하지만 클라이언트는 이 부분을 놓치기 쉬워서 예측할 수 없는 성능 문제를 야기할 수 있다.

다음 두가지 방법으로 자원을 회수할 수 있다.

하지만 첫번째 방식인 `try-finally`는 더이상 권장하지 않고, `try-with-resources` 방식을 권장한다.

두가지를 비교해보자.

#### **<span style="color:#ef5369">try-finally 방식</span>**

기존 `try-finally`방식은 자원을 제대로 닫힘을 보장하는 수단으로 사용되었다.

```java
//close try-finally 감싸기
public String readLineOnce(String path){
    String line = "";
    BufferedReader br = null;
    try{
        br = new BufferedReader(new FileReader(path));
        line = br.readLine();
    } catch (Exception e) {
        throw new RuntimeException(e);
    } finally {
        if(br != null){
            try{
                br.close();
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
    }
    return line;
}
```

보다 시피 `close()`를 사용하기 위해 `finally`의 구문에서도 마찬가지로 `try-finally`로 감싸줘야한다.

```java
//close try-finally 감싸기
public String readLineOnce(String path) throw IOException{
    String line = "";
    BufferedReader br = null;
    try{
        br = new BufferedReader(new FileReader(path));
        line = br.readLine();
    } catch (Exception e) {
        throw new RuntimeException(e);
    } finally {
        br.close();
    }
    return line;
}
```

또는 그냥 예외를 발생 시키게 된다면, 첫번째 예외 대신 `close()`에 해당하는 예외가 발생하게 되어 첫번째 예외는 무시하게 될 것이다.

---

#### **<span style="color:#ef5369">try-with-resources 방식</span>**

해당 문제는 Java 7 부터 생긴 `try-with-resources`로 해결이 가능하다.

이 구조를 사용하기 위해선 `close()`가 필요한 자원이 `AutoCloseable` 인터페이스를 구현해야한다.

```java
//단일 적용
public String readLineOnce(String path) throws IOException{
    String line = "";
    try(BufferedReader br = new BufferedReader(new FileReader(path))){
        line = br.readLine();
    }
    return line;
}
```
```java
//복수 적용
public String readLineOnce(String path) throws IOException{
    String line = "";
    try(FileReader fr = new FileReader(path);
        BufferedReader br = new BufferedReader(fr)){
        line = br.readLine();
    }catch(Exception e){
        e.printStackTrace();
    }
    return line;
}
```

`try-with-resources`를 사용하게 되면 짧고 읽기 수월하다.

그리고 기존 `try-finally`는 중첩 `try`문을 사용했어야하지만 지금은 중첩을 사용하지 않고도 다수의 예외 처리가 가능하다.

추가적으로 말하자면, `try-with-resources`도 `try-finally`와 같이 `catch`를 사용해 예외처리가 가능하다.

---

결론적으로 꼭 회수해야하는 자원이 있다면 `try-finally` 대신 `try-with-resources`를 사용하여 개발하자.

지저분한 코드를 줄여주고, 더욱 깔끔하고 쉽게 자원 회수와 예외처리가 가능해질 것이다.