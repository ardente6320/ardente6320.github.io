---
title:  "DB에 존재하는 테이블 명 조회 및 컬럼 조회"
date:   2022-12-01 23:09:13+0900
categories: [PostgreSQL]
tags: [PostgreSQL, sql]
---
<br>

**<span style="color:#ef5369">PostgreSQL</span>**에서 DB에 존재하는 테이블과 컬럼을 조회하려면 특정 테이블을 조회해야한다.

조회 방법에 앞서 **<span style="color:#ef5369">`information_schema`</span>**에 대해 알아야한다.

<br>

#### **<span style="color:#ef5369">`information_schema`란?</span>**

DB에 속한 데이터들의 정보를 담고 있는 메타데이터이다.

스키마 정보, 테이블 정보, 컬럼 정보 등등 가지고 있다.

---

#### **<span style="color:#ef5369">테이블 조회</span>**

<br>

##### **<span style="color:#ef5369">1. `pg_stat_user_tables` 사용하여 테이블 조회하기</span>**

```sql
SELECT schemaname  -- 스키마 명
     , relname     -- 테이블 명
FROM pg_stat_user_tables
```

`pg_stat_user_tables` 테이블을 조회하면 사용자가 접근할 수 있는 모든 데이터베이스와 테이블 내역을 보여준다.

<br>

##### **<span style="color:#ef5369">2. `information_schema` 사용하여 테이블 조회하기</span>**

```sql
select table_schema -- 스키마 명
     , table_name   -- 테이블 명
from information_schema.tables
where 1=1
  and table_schema = '조회하려는 스키마 명'
  and table_type = 'BASE TABLE'
```

`information_schema.tables`를 조회하면 스키마와 테이블 명을 확인 할 수 있다.

여기서 `BASE TABLE`은 `create table` 을 통해 생성된 테이블이다.

---

#### **<span style="color:#ef5369">컬럼 조회</span>**

<br>

##### **<span style="color:#ef5369">1. information_schema 사용하여 컬럼 조회하기</span>**

```sql
select table_schema -- 스키마 명
     , table_name   -- 테이블 명
     , column_name  -- 컬럼 명
     , data_type    -- 데이터 타입
     , is_nullable  -- nullable 값(YES / NO)
from information_schema.columns
where 1=1
  and table_schema = '스키마 명'
  and table_name = '테이블 명'
```

`information_schema.columns`를 조회하면 해당 테이블에 대한 컬럼 정보를 확인 할 수 있다.