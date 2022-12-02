---
title:  "LEAST,GREATEST를 활용하여 최솟값, 최댓값 찾기"
date:   2022-12-01 23:30:02
categories: [PostgreSQL]
tags: [PostgreSQL, sql]
---
<br>

#### **<span style="color:#ef5369">LEAST, GREATEST 함수란?</span>**

LEAST 함수는 최소값, GREATEST 함수는 최대값을 반환하는 함수이다.

<br>

#### **<span style="color:#ef5369">MIN, MAX 함수와의 차이점</span>**

MIN, MAX는 여러 행(row)중에 해당 열(column)의 최소, 최대를 반환한다.

LEAST, GREATEST는 여러 열(column)에서 최소, 최대를 반환한다.

<br>

#### **<span style="color:#ef5369">함수 사용방법</span>**

LEAST, GREATEST함수는 인자를 N개 가질 수 있고, 해당 인자에서 최소, 최대값을 반환한다.

인자의 데이터 타입은 모두 동일해야한다.


`LEAST("값1", "값2", "값3", "값4", "값5", "값6", ...)` 인자값 중 **최솟값**을 반환

`GREATEST("값1", "값2", "값3", "값4", "값5", "값6", ...)` 인자값 중 **최댓값**을 반환

---

#### **<span style="color:#ef5369">예시</span>**

<br>

##### **<span style="color:#ef5369">LEAST 활용하여 최솟값 반환 </span>**

```sql
SELECT LEAST(10,20,30,40,50,60) FROM DUAL

--result : 10
 ```

##### **<span style="color:#ef5369">GREATEST 활용하여 최댓값 반환</span>**

```sql
SELECT GREATEST(10,20,30,40,50,60) FROM DUAL

--result : 60
```

---

#### **<span style="color:#ef5369">주의 사항</span>**

<br>

##### **<span style="color:#ef5369">1. NULL이 포함되어 비교하게 되는 경우</span>**

LEAST,GREATEST 함수 사용 시, NULL을 결과로 반환하게 된다. 

따라서 NULL을 다른 값으로 치환하여 사용해야한다.

<br>

- LEAST에서 NULL 비교

```sql
--LEAST 에서 NULL 비교
SELECT LEAST(10,20,NULL) FROM DUAL

--result : NULL

--대안
SELECT LEAST(10,20,NVL(NULL,0)) FROM DUAL

--result : 0
```

- GREATEST에서 NULL 비교

```sql
--GREATEST 에서 NULL 비교
SELECT GREATEST(10,20,NULL) FROM DUAL

--result : NULL

--대안
SELECT GREATEST(10,20,NVL(NULL,0)) FROM DUAL

--result : 20
```

<br>

##### **<span style="color:#ef5369">2. 데이터 타입이 다른 경우</span>**

데이터 타입이 다른 인자가 섞여 있을 경우 오류가 발생한다.

**다만, 숫자와 문자형 숫자인 경우에는 반환을 한다.**

<br>

- 숫자와 문자 비교

```sql
--숫자와 문자 비교
SELECT GREATEST(10,20,'test') FROM DUAL;
SELECT LEAST(10,20,'test') FROM DUAL;

--result : SQL Error [22P02]: ERROR: invalid input syntax for integer: "test"
```

- 숫자와 문자형 숫자 비교

```sql
--숫자와 문자형 숫자 비교
SELECT GREATEST(10,20,'30') FROM DUAL

--result : 30

SELECT LEAST(10,20,'30') FROM DUAL

--result : 10
```