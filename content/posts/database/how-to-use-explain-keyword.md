---
author: Han
title: Explain 사용에 대한 이해
date: 2021-06-16
description: A brief guide to how to use explain
tags: ['mysql']
ShowToc: true
---


# Explain 이란?
- 기본적으로 `SELECT`, `INSERT`, `DELETE`, `REPLACE`, `UPDATE` 쿼리문의 실행플랜을 분석하는 데 사용하는 키워드.
- 해당 쿼리를 실제 실행하는 것은 아니고, 데이터베이스에게 어떻게 실행할 건지 계획을 받아보는 방법임.
- 아래와 같이 사용함.

```SQL
EXPLAIN SELECT * FROM foo WHERE foo.bar = 'infrastructure as a service' OR foo.bar = 'iaas';
```

# Explain 결과 이해하기

### table
- 어떤 테이블에 접근하고 있는가

### id
- SELECT에 붙은 번호..
- 즉 SELECT 를 몇번이나 실행하는지..?
- 해당 쿼리가, subquery, union 쿼리를 포함하고 있지 않다면, 항상 1임.

### select_type
- SELECT Quey의 타입
- **SIMPLE** -> 서브쿼리나 유니온 쿼리 없이 실행된 SELECT 쿼리
- **SUBQUERY**
- **UNION**
- **DERIVED**
- [MySQL's Doc](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_type) , 여기 참고할 것.

### partitions
- 해당 테이블이 파티셔닝 되어있을 경우, 사용되는 필드.
- `NULL` 은 해당 쿼리에서 사용되는 테이블이 파티셔닝 되지 않았을을 의미.

### type
- 어떻게 해당 테이블에 접근하고 있는가를 나타낸 필드.
- 이 필드는, 해당 쿼리의 효율성을 판단하는 데 가장 중요한 필드 임
- 주의할 타입
    1. ALL -> 전체 행 스캔, 테이블에 존재하는 모든 데이터 접근.
    2. index -> 인덱스 풀 스캔, 인덱스를 처음 부터 끝까지 검색하는 경우
    3. ref_or_null -> 조인할 경우, Primary Key 혹은 Unique Key가 아닌 Key로 매칭되고 , null이 추가적으로 검색되는 경우..

### possible_keys
- 이용 가능성이 있는 인덱스의 목록

### key
- 위 possible_keys 중, 실제 사용하겠다고 선택된 인덱스

### rows
- 위 나열된 접근방식을 통해서 몇 행을 가져왔는지 의미.

### filtered
- 가져온 rows가 `WHERE` 을 통해 얼마나 필터되었는지 의미
- 퍼센테이지
- 20... 이면 20%가 필터되고, 나머지 80%정도가 남을 예정이라는 의미./
-  참고
   - https://stackoverflow.com/questions/22969672/mysql-explain-extended-filtered-column-obviously-its-not-a-percentage

### extra
- 쿼리 실행 플랜이 가지고있는 추가적인 정보들..
1. `Using Where` : 가져온 rows를 `WHERE` 을 이용하여 필터될 예정이라는 으미ㅣ.
2. `Using Index` : 인덱스를 이용하여, rows 필터하는 것을 의미. index only scan


> 참고
- https://www.exoscale.com/syslog/explaining-mysql-queries/#:~:text=In%20MySQL%2C%20EXPLAIN%20can%20be,as%20a%20service'%20OR%20foo.
- https://cheese10yun.github.io/mysql-explian/
- https://www.eversql.com/mysql-explain-example-explaining-mysql-explain-using-stackoverflow-data/
- https://nomadlee.com/mysql-explain-sql/
- https://www.sitepoint.com/using-explain-to-write-better-mysql-queries/#:~:text=Extra%20%E2%80%93%20contains%20additional%20information%20regarding,may%20indicate%20a%20troublesome%20query.