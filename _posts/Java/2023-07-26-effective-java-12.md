---
title:  "Effective Java-ITEM 12 toString을 항상 재정의하라"
date:   2023-07-26 07:57:48+0900
categories: [Java]
tags: [Java, Effective Java, toString]
---
<br>

`toString`은 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다.

```java
Person A = new Person(30,"M","123456");

System.out.println(A); //Person@56768054
```

위 코드 처럼 `Object`의 기본 `toString`메소드는 `클래스_이름@16진수로_표시한_해시코드`를 반환한다.

간결하지만 `age=30,gender=M,serialNumber=123456`처럼 직접 알려주는 형태가 훨씬 유익한 정보를 담고 있다.

<br>

**`toString`을 잘 구현한 클래스는 디버깅하기 쉬워진다.**

<br>

`Person`클래스의 `toString`은 다음과 같이 재정의 하면 된다.

```java
class Person{
    int age;
    String gender;
    String serialNumber;
    
    @Override
   public String toString(){
        return String.format("age=%d,gender=%s,serialNumber=%s",age,gender,serialNumber);
    }
}
```

---

#### **<span style="color:#ef5369">`toString`재정의할 때 주의할 점</span>**

**1. 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다.** 

- 보안적 이슈가 있는 주요 정보는 노출하는 것은 좋지않으니, 반드시 필요한 정보를 노출하자.
- 객체가 거대한 경우나 객체의 상태가 문자열로 표현하기 적합하지 않은 경우는 요약하여 노출하도록 하자.
   ```
   나이=30,성별=남자 ...
   ```

<br>

**2. 포맷을 명시하기로 했으면, 명시한 포맷에 맞는 문자열과 객체를 상호전환 할 수 있는 정적 팩터리나 생성자를 함께 제공하면 좋다.**

다음 휴대폰 번호 클래스와 같이 상호 전환이 가능한 메소드가 있으면 좋다.

```java
class PhoneNumber{
   private int areaCode;
   private int prefix;
   private int lineNum;
    
   private static final Pattern pattern = Pattern.compile("^\\d{3}-\\d{4}-\\d{4}$");
   
   public PhoneNumber(int areaCode, int prefix, int lineNum){
       this.areaCode = areaCode;
       this.prefix = prefix;
       this.lineNum = lineNum;
   }
   
   @Override
   public String toString() {
      return String.format("%03d-%04d-%04d", areaCode, prefix, lineNum);
   }

   public static PhoneNumber parse(String phoneNumber) {

      if(!pattern.matcher(phoneNumber).find()) {
         throw new UnknownFormatConversionException("Invalid Pattern[%03d-%04d-%04d] : "+phoneNumber);
      }

      String[] numbers = phoneNumber.split("-");
      return new PhoneNumber(Integer.parseInt(numbers[0]),Integer.parseInt(numbers[1]),Integer.parseInt(numbers[2]));
   }
}
```

**<span style="color:#ef5369">단점도 존재한다.</span>**

포맷을 한번 명시하게 되면 평생 그 포맷에 얽메이게 된다.

만약 향후 릴리스에서 포맷을 바꾼다면 기존에 사용하던 코드와 데이터는 엉망이 될 것이다.

반대로 포맷을 명시하지 않는다면 향후 릴리스에서 정보를 더 넣거나 포맷을 개선할 수 있는 유연성을 얻게 된다.

<br>

**3. 포맷을 명시하는 아니든 의도는 명확히 밝혀야 한다.**

포맷을 명시하려고 한다면 정확히 작성해야한다.

```java
/**
 * 포맷을 명시한 경우
 * 해당 인스턴스는 사람 정보에 대한 표현을 반환한다.
 * 이 문자열은 사람의 나이, 성별, 주민번호를 나타낸다.
 * 
 * 형태는 다음과 같다. age=XXX,gender=Y,serialNumber=ZZZZZZZZZZZZZ
 * XXX는 숫자 형태의 나이, Y는 성별(M[남자],F[여자]), ZZZZZZZZZZZZZ는 주민번호를 나타낸다.
 * 
 * 나이의 경우 너무 적어 자릿수를 채울 수 없다면 앞에서 부터 0으로 채워 나간다.
 * 예를 들어 나이가 9살인 경우 age=009로 표시한다.
 */
@Override
public String toString(){
    return String.format("age=%03d,gender=%s,serialNumber=%s",age,gender,serialNumber);
}
```

포맷을 명시하지 않는다면 다음과 같이 작성할 수 있다.

```java
/**
 * 포맷을 명시하지 않은 경우
 * 해당 인스턴스는 사람에 대한 대략적인 설명을 반환한다.
 * 다음은 이 설명의 일반적인 형태이나, 상세 형식은 정해지지 않았으며 향후 변경 될 수 있다.
 * 
 * "이름=웨이드,나이=30,성별=남자"
 */
@Override
public String toString(){
    ...
}
```

<br>

**4. `toString`의 반환값에 포함된 정보를 얻어올 수있는 API를 제공하자.**

예를 들어 `Person`클래스는 나이, 성별, 주민번호에 대한 접근자를 제공해야한다.

```java
class Person{
    private int age;
    private String gender;
    private String serialNumber;
    
    public Person(int age, String gender, String serialNumber){
        this.age = age;
        this.gender = gender;
        this.serialNumber = serialNumber;
    }
    
    public int getAge(){
        return age;
    }
    
    public String getGender(){
        return gender;
    }
    
    public String getSerialNumber(){
        return serialNumber;
    }

    @Override
    public String toString(){
        return String.format("age=%03d,gender=%s,serialNumber=%s",age,gender,serialNumber);
    }
}
```

위 코드와 같이 접근자를 제공하지 않는다면 `toString`의 반환값을 파싱할 수밖에 없다.

성능은 나빠지고, 향후 포맷을 바꾸면 시스템이 망가지는 결과를 초래할 수 있다.

---

모든 Concrete Class에서 `Object`의 `toString`을 재정의 해야한다.

만약 상위 클래스에서 이미 알맞게 했다면 상관없다.

시스템 디버깅을 쉽게 하기 위해서, 그리고 유용한 정보를 제공하기 위해서라도 재정의를 하도록 하자.
