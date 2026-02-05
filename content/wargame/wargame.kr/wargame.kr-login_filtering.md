---
title: "[Wargame.kr] login filtering 문제풀이"
date: 2018-10-23
draft: false
category: [wargame]
subcategories: [wargame.kr]
tags: [Write-up, Wargame.kr]
---

매일 한 문제씩 푼다고 했는데 작심삼일도 못했다.
갑자기 일이 많아져서 뭔가 여유가 없다.
그래도 문제는 꾸준히 풀어야하니 오늘 잠깐 짬 났을때 하나 풀어 보았다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
I have accounts. but, it's blocked.

can you login bypass filtering?
```

문제에 접속하면 로그인 화면이 하나 있고 소스코드를 볼 수 있게 되어있다.
소스코드는 다음과 같다.  

```php
<?php

if (isset($_GET['view-source'])) {
    show_source(__FILE__);
    exit();
}

/*
create table user(
 idx int auto_increment primary key,
 id char(32),
 ps char(32)
);
*/

 if(isset($_POST['id']) && isset($_POST['ps'])){
  include("../lib.php"); # include for auth_code function.

  mysql_connect("localhost","login_filtering","login_filtering_pz");
  mysql_select_db ("login_filtering");
  mysql_query("set names utf8");

  $key = auth_code("login filtering");

  $id = mysql_real_escape_string(trim($_POST['id']));
  $ps = mysql_real_escape_string(trim($_POST['ps']));

  $row=mysql_fetch_array(mysql_query("select * from user where id='$id' and ps=md5('$ps')"));

  if(isset($row['id'])){
   if($id=='guest' || $id=='blueh4g'){
    echo "your account is blocked";
   }else{
    echo "login ok"."<br />";
    echo "Password : ".$key;
   }
  }else{
   echo "wrong..";
  }
 }
?>
<!DOCTYPE html>
<style>
 * {margin:0; padding:0;}
 body {background-color:#ddd;}
 #mdiv {width:200px; text-align:center; margin:50px auto;}
 input[type=text],input[type=[password] {width:100px;}
 td {text-align:center;}
</style>
<body>
<form method="post" action="./">
<div id="mdiv">
<table>
<tr><td>ID</td><td><input type="text" name="id" /></td></tr>
<tr><td>PW</td><td><input type="password" name="ps" /></td></tr>
<tr><td colspan="2"><input type="submit" value="login" /></td></tr>
</table>
 <div><a href='?view-source'>get source</a></div>
</form>
</div>
</body>
<!--

you have blocked accounts.

guest / guest
blueh4g / blueh4g1234ps

-->
```

이 문제 풀면서 느낀건데 뭔가 난 아직 문제를 많이 안풀어봐서 그런건지, 아니면 센스가 부족한건지...
다른 시각이 필요한데 그걸 잘 못본다 ㅠㅠ
일단 처음에는 SQL Injection 문제인 줄 알았다.

인터넷을 찾아보니 `mysql_real_escape_string` 함수를 bypass 할 수 있다는 글이 있어서 bypass 해서 다른 계정으로 로그인을 하면 되는 줄 알았다.
그런데 등잔밑이 어둡다더니 취약점은 `if` 구문이었다.
`if` 문을 살펴보면 다음과 같다.  

```php
  if(isset($row['id'])){
   if($id=='guest' || $id=='blueh4g'){
    echo "your account is blocked";
   }else{
    echo "login ok"."<br />";
    echo "Password : ".$key;
   }
  }else{
   echo "wrong..";
  }
```

만약 쿼리를 실행 한 결과가 있다면, 해당 `id`가 `guest` 혹은 `blueh4g` 이라면 `your account is blocked`을 출력 해 준다.
그런데 이 때, `==`를 사용하며 비교 대상은 쿼리 결과로 나온 `$row['id']`가 아니라 내가 입력 한 `$id`이다.

MySQL에서는 대소문자를 구분하지 않기 때문에 만약 내가 `$id`의 값으로 `Guest`를 입력 하더라도 DB에 `guest`가 있다면 `id`가 `guest`인 행을 얻어낼 수 있다.
그렇지만 `if($id=='guest' || $id=='blueh4g')` 구문에서 `$id`는 내가 입력 한 `Guest`일 테니, 조건문에 들어가지 않을 수 있다.
SQL Injection인 줄 알았는데!!!  

`ID` 자리에 `Guest`를, `PW` 자리에 `guest`를 입력하면 문제를 풀 수 있다.
너무 허무하다... ㅠㅠ  

![](/images/wargame.kr/login_filtering/login_filtering_01.png)

```plain
FLAG : 9783333beeb385184e9def185bc99ec81d72254b
```