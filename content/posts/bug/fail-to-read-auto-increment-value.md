---
author: Han
title: Failed to read auto-increment value from storage engine
date: 2021-08-18
tags: ['bug', 'mysql']
ShowToc: false
---

# 1. 버그 내용
- 내용만 보면, `storage engine` 에서 `auto-increment` value를 읽는 데 실패했다는 오류.
- 아마도 해당 값이 잘못된 값을 읽어오려고 해서 발생하는 버그이지 않을까 싶음.

# 2. 검색
- [Failed to read auto-increment value from storage engine 해결 방법](https://jojoldu.tistory.com/417) 이 바로 나옴.
- 현재 테이블의 `auto-increment` 상태값이 잘못 설정되어 있어서, 오류가 발생하는 것.
    - 즉 현재 row 갯수 + 1 이 아닌, 다른 값(0..) 이 들어가 있는 현상이 발생하기도 하는듯.
- 테이블 상태값 확인
    ```mysql
    show table status like '테이블명'
    ```
- 강제로 최신 id값을 가진 row를 추가를 통해, 테이블 상태값 변경
    ```mysql
    insert into table (pk필드) values (최신 PK +1)
    ```
- 그렇지만 현재 상황하고는 안 맞는듯 하다. 
    - 왜? 
    - 테이블 상태값을 확인했을 때, 잘못된 값이 설정되어 있는 것 처럼 보이지 않는다.
- 그것보다, 해당 `id` 값이 42..억이 넘는다
    - 아마도 테이블에 id 타입이 `int`로 선언되어 있고, 최대치에 도달했기 때문에 발생하지 않을까?


## 2.1 INT vs BIGINT
![image](https://user-images.githubusercontent.com/22140570/129900275-7f13192d-433e-4ca3-a899-34055acf6a4a.png)
- 위 이미지에서도 볼 수 있듯이, unsigned int 데이터 타입의 최대 value는 약 43억.
- `auto-increment` 에 저장되는 값이, 이 한계치를 초과했기 때문에 발생한 문제로 여겨진다.
- [참고](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)

# 3. 해결 방법
```txt
1. 문제 발생하는 테이블(A)을 카피하여 새로운 테이블(B) 생성 
2. 새로운 테이블(B)의 id column 타입 변경
3. 새로운 테이블(B)의 이름을 (A) 로 변경
4. A 테이블 드랍
```

- 이 방법이 production 환경에서, 할 수 있는 최선이지 않을까..
- mysql int to bigint로 더 검색해보자
- [참고](https://dba.stackexchange.com/questions/95740/alter-primary-key-column-from-int-to-bigint-in-production-mysql-5-6-19a)

# 4. 정리
- [MySQL에서 DB스키마 작성시 주의할 점들](https://novemberde.github.io/database/2019/06/10/MySQL-Desgin-cheatsheet.html)
- 테이블을 만들기 전에, 이 포스팅을 한번 참조해보면 좋을 것 같음.

> ID값은 BIGINT를 추천한다. 나중에 서비스가 커져 INT를 BIGINT로 바꾸는 경우도 생각보다 만만치 않다. 고려할 점이 생긴다. 마음 편히 BIGINT로 ID를 지정하여 추후에 있을지도 모를 업무를 줄여야 한다.

- 특히 테이터 삽입이 잦은 테이블이라면, 미리 id의 데이터 타입을 BIGINT로 잡고 가는 것도 나쁘지는 않을 것 같다는 생각이 듬. 

