---
title: "[Lord of SQL Injection] LoS - vampire 문제풀이"
date: 2018-07-28
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번 문제도 역시 간단하게 풀 수 있었다.
일단 소스코드가 짧으니 문제가 더 간단하게 느껴지는 것 같다.  

<!--more-->

코드는 다음과 같다.  

```php
----------------------------------------------------------------
query : select id from prob_vampire where id=''
----------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/\'/i', $_GET[id])) exit("No Hack ~_~");
  $_GET[id] = strtolower($_GET[id]);
  $_GET[id] = str_replace("admin","",$_GET[id]); 
  $query = "select id from prob_vampire where id='{$_GET[id]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id'] == 'admin') solve("vampire"); 
  highlight_file(__FILE__); 
?>
```

처음에 문제를 보고 `troll` 문제랑 같은 문제인 줄 알았다.
그런데 이번 문제에서는 `ereg()` 함수가 사라지고 `strtolower()`과 `str_replace()` 함수가 생겼다.
함수 이름에서도 알 수 있다시피 `strtolower()` 함수는 문자열의 대문자를 소문자로 바꾸어주는 함수이고, `str_replace()` 함수는 특정 문자열을 내가 지정한 문자열로 바꾸어 주는 함수이다.

문제에서는 `GET` 방식으로 받아 온 값에서 `admin`이라는 문자를 `공백`으로 바꾸어 준다.
그런데 이 `str_replace()` 함수에는 한가지 단점(?)이 있는데, 전체 문자열 전체를 한번만 검사하고 재검사는 하지 않는다는 것이다.
만약에 `str_replace("admin", "", $var)`라고 구현한다면 아래와 같이 문자열이 있을 경우 결과는 다음과 같다.  

```php
#before

$var = "admin phpmyadmin administrator."

#after

$var = " phpmy istrator."
```

그런데 만약 `adadminmin` 라는 문자열이 있다면 가운데의 `admin`만 필터링 되어 사라지고 앞의 `ad`와 뒤의 `min`이 합쳐져 다시 `admin`이라는 단어를 만들 수 있게 된다.
역시나 주소창에 아래처럼 입력했더니 문제를 풀 수 있었다.  

```plain
?id=adadminmin
```

--- 

```php
------------------------------------------------------------------------
query : select id from prob_vampire where id='admin'
------------------------------------------------------------------------

VAMPIRE Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/\'/i', $_GET[id])) exit("No Hack ~_~");
  $_GET[id] = strtolower($_GET[id]);
  $_GET[id] = str_replace("admin","",$_GET[id]); 
  $query = "select id from prob_vampire where id='{$_GET[id]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id'] == 'admin') solve("vampire"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
VAMPIRE Clear!!
```