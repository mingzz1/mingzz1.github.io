---
title: "[Database-DB2] DB2 Query 정리"
date: 2020-01-06
draft: false
category: [development]
subcategories: [database]
tags: [Database, DB2]
---

DB2의 데이터베이스 및 테이블을 생성하고, 데이터를 추출하는 쿼리는 다음과 같다.  

<!--more-->

일단 DB2에서는 쿼리를 2가지 방법으로 실행할 수 있다.

```sh
# 그냥 Shell에서 실행
$ db2 "Query"

OR

# Shell에서 db2 콘솔 실행
$ db2
db2 => Query # db2 콘솔에서 쿼리 실행
```

예를 들면 다음과 같다.  

```sh
$ db2 "select * from table;"

OR

$ db2
db2 => select * from table;
```

어느 방법이든 쿼리와 결과는 동일하기 때문에 편한 방식으로 사용하면 될 것 같다.  

* 데이터베이스 생성  

```mysql
create database DB_NAME
```

* 테이블 생성

```mysql
create table DB_NAME.TABLE_NAME(COL_1 char(30) not null, COL_2 varchar(100) not null);
```

* insert

```mysql
insert into DB_NAME.TABLE_NAME(COL_1, COL_2, ...) values('VAL1_1', 'VAL1_2', '...'), ('VAL1_1', 'VAL1_2', '...'), (...);
```

* select

```mysql
select * from DB_NAME.TABLE_NAME where CONDITION;
```

---

#### 참고사항
* where절 사용 시, 문자는 작은따옴표(')로 묶어야 하고, 조건은 대소문자를 구분한다.

```mysql
select * from table where col='CONDITION';
```

이 때, 'CONDITION'은 대소문자를 구분하기 때문에 `col='condition'`과 `col='CONDITION'`은 다른 결과를 얻는다.

* 전체 데이터베이스 리스트 출력

```mysql
list database directory
```

* 전체 테이블 리스트 확인

```mysql
select * from sysibm.systables;
```

이 때, `name` 컬럼은 `테이블 명`, `creator` 컬럼은 `데이터베이스 명`이다. 따라서 아래와 같이 사용하면 특정 데이터베이스 내의 테이블 명만 추출할 수 있다.  

```mysql
select name from sysibm.systables where creator='DB_NAME';
```

* 특정 개수만큼 select

```mysql
select * from TABLE_NAME fetch first NUM rows only;
```

이렇게 하면 각 테이블의 `첫 행부터 NUM개의 행`을 select 한다. 만약 `마지막 행부터 특정 개수의 행을 추출`하고 싶다면, 특정 column을 기준으로 `order by desc`한 후 `fetch first NUM rows only`를 사용해야 한다.  
(IBM 공식 답변에 따르면 테이블의 마지막 행부터 특정 개수의 행을 추출하는 쿼리는 따로 만들어져 있지 않다고 함)
